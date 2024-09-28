---
title: "How we implemented a better approach than certificate transparency and pinning"
date: 2024-09-28 00:00:00 +0000
categories: [TLS]
tags: [TLS, SSL, HostnameVerifier]
---

A few days ago I was reading [this](https://blog.cloudflare.com/why-certificate-pinning-is-outdated/) article by Cloudflare which brought back some memories of how we dealt with the issue of certificate pinning on Android. It motivated me to write this blog about how we implemented a hybrid approach which is fast and secure at the same time. While also sharing the drawbacks of each approach.

Before we begin, I want you to remember that the Fi app uses GRPC as its core network stack. However, these solutions can be applied to other types of network calls as well. However, the implementation steps may vary.

With Vanilla TLS, it's easy to do a Man In The Middle (MITM) attack. Thus we wanted something stronger than that.

### Certificate Transparency (CT)
Certificate Transparency is an additional step performed over the standard TLS verification, like verifying the Hostname name and root CA etc. When Certificate Transparency is performed, it makes a network call to a separate log server with the Signed Certificate Timestamp (SCT) embedded in the certificate. 

You can learn how certificate transparency works [here](https://certificate.transparency.dev/howctworks/).
{: .notice--info}

Fi also being a payments app, people would use the app at the storefront to make payments. The internet connectivity here would likely be very flaky or poor. In such cases making any additional calls on the network leads to extra time taken in app launch. After doing a lot of analysis, we found that CT was adding up to an extra 2 seconds even before a connection was established with the server.

Thus we started looking for something more efficient while also secure. That's how we decided to try certificate pinning.

### Certificate Pinning (Pinning)
Most of the time when we pin, we are not pinning the certificate itself, but the hash of the public key, found in the certificate. The advantage here is that we can rotate the certificate but may keep the public key the same. However, this is not recommended. Thus the name Certificate Pinning is incorrect and should have been Public Key pinning.
{: .notice--info}

We pinned the public key using a network security config. Which is the recommended way to do it according to Android's [docs](https://developer.android.com/privacy-and-security/security-config#CertificatePinning).

Whenever we do pinning, it always comes at the cost of flexibility. The flexibility to change the public key before the expiry of the certificate or unexpected leak of the private key.

Because the certificate rotation bit is very complex. If we miss any step during key rotation, then we block several users out of the app. Since pinning is also done within an XML file, it doesn't allow us to implement any logic. These were some of the big factors for us not to go with just vanilla pinning. This made us look for alternatives.

### Hybrid approach
We came up with a hybrid approach that combined both CT and pinning. The drawback of CT is that it was slow, but flexible in terms of certificate rotation. The problem with pinning is that it's faster even when on a slow network but not flexible when rotating the keys. Thus we combined both to get the goodness of each.

We implemented a mechanism in our `HostNameVerifier` where firstly we did the traditional check performed by TLS and then verified the pins. This would pass in most of the cases. However, when we have just rotated the certificate, the older app versions with older pins will fail the pinning check. When it fails, we fall back to CT to verify the certificate, over the network. This is also effective in blocking the unsolicited MITM attacks.

```kotlin
internal class PinningOrCertificateTransparencyHostNameVerifier(
    private val pins: Map<Host, Set<PublicKeySha256>>,
    private val certificateTransparencyHostNameVerifier: HostnameVerifier
) : HostnameVerifier {

    @SuppressLint("BadHostnameVerifier")
    override fun verify(hostname: String, session: SSLSession): Boolean {
        // We should call verifyOkHostname first and if it passes
        // call pinning first or certificate transparency either has to return true
        // for the connection to be considered secure.
        return verifyOkHostname(hostname, session)
            && (verifyPublicKeyPinning(hostname, session)
            || verifyCertificateTransparency(hostname, session))
    }

    private fun verifyOkHostname(hostname: String, session: SSLSession): Boolean {
        val okHttpResult = OkHostnameVerifier.verify(hostname, session)
        Timber.i("OkHostnameVerifier result: $okHttpResult")
        return okHttpResult
    }

    private fun verifyPublicKeyPinning(hostname: String, session: SSLSession): Boolean {
        // We try and find a pin for the hostname, else return false
        val setOfPins = pins[Host(hostname)] ?: kotlin.run {
            Timber.w("No pin found for $hostname")
            return false
        }
        val leafCertificate: Certificate = session.peerCertificates[0]
        val sha256: String = leafCertificate.publicKey.encoded.sha256String()
        val pin = PublicKeySha256(sha256)
        val isPinVerificationSuccessful = setOfPins.contains(pin)
        Timber.i("VerifyPublicKeyPinning result: $isPinVerificationSuccessful")
        return isPinVerificationSuccessful
    }

    private fun verifyCertificateTransparency(hostname: String, session: SSLSession): Boolean {
        val ctResult = certificateTransparencyHostNameVerifier.verify(hostname, session)
        Timber.i("CertificateTransparency result: $ctResult")
        return ctResult
    }
}

@JvmInline
internal value class Host(val host: String)

@JvmInline
internal value class PublicKeySha256(val pin: String)
```

<figure class="align-center">
  <img src="/assets/images/hybrid_verification.png" alt="Hybrid verification" style="width:100%;height:100%;">
</figure>


This of course comes with some drawbacks
1. An app would make many different forms of connection in different areas of the app. Eg: We make the API call to load the page, which shows a Terms and Conditions hyperlink. When clicked on it, it opens a web view which shows the Terms and conditions. Here we are making a REST API call to load the page and then loading a Webpage, which will form its new connection to the server even when the endpoints remain the same. In such cases having a centralised place like the Android's recommended way of pinning would be the best. Since it will verify the same pin for any connection established to the domain by the app. However, when we use the hybrid approach, the code to verify the pin lives within a `HostNameVerifier`, which we will have to individually apply on any new connection established by the app.
2. When the certificate is rotated, older versions of the app will fall back to CT. The users who are accustomed to the shorter load time will see this increased load time, which is not an ideal experience for any user. However, once in the app, we can nudge them to update the app to the latest version with pins for new certificate.


### Summary
* Certificate Transparency
  * Pro
    1. No extra handling is needed when the certificate is rotated, as some verification happens on the network.
  * Con
    1. Very slow when on an unstable internet connection
* Certificate pinning or public key pinning
  * Pro
    1. Faster than CT as all the checks are performed locally on the device.
    2. Can be configured in one place and it will take effect across the app.
  * Con
    1. Very tricky to rotate the certificate. If not planned and executed properly We might lock some or all users out of the app forever.
* Hybrid Approach (Combination of pinning and CT)
  * Pro
    1. Speedy verification of pinned certificate
    2. Gets the flexibility of CT when certificates are rotated
  * Con
    1. Cannot be configured centrally in one place
    2. Takes time to establish the connection as pin verifications will fail and it will fall back to CT.