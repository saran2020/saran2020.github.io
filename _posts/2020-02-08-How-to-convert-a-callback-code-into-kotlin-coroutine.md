---
title: How I renamed an entire library already in production
categories:
  - Kotlin
tags:
  - Android
  - Android-Studio
  - Kotlin
  - Co-routines
  - Callback
  - Java
---

As you already know, Kotlin Co-routines turns a callback based code block into sequential code. Let me show you an example for the people who doesn't know already.

```
    val getUserCall = apiService.getUser(15)
	getUserCall.enqueue(object: Callback<User> {
		override fun onResponse(call: Call<User>, response: Response<user>) {
            doSomeThing(response.body())
		}

		override fun onFailure(call: Call<User>, t: Throwable) {
			t.printStackTrace()
		}
	})
```

This is how a typical callback based code would look like in Kotlin. But, aren't we talking about co-routines, why do we still have callbacks?

Ok let me show you the above example using co-routines

```
	launch {
		try {
    		val user = apiService.getUser(15)
    		doSomeThing(user)
		} catch(t: Throwable) {
			t.printStackTrace()
		}
	}
```

Cool!! but here we assume that `apiService.getUser(15)` is a function which supports co-rotines. However often that's not the case, becasue usually we are using a framework or libraries, thats written on/or for Java. In such case, we can't avoid using callbacks. 

In such cases what we can do though is convert these callbacks into Kotlin co-routines. Lemme show you how!!

Let's take the previous example of getting a user from a api call.

```
	launch {
		try {
    		val user = getUser(apiService, 15)
    		doSomeThing(user)
		} catch(t: Throwable) {
			t.printStackTrace()
		}
	}

	suspend fun getUser(apiService: Service, id: Int) = suspendCoroutine<User> { continuation ->
		val getUserCall = apiService.getUser(15)
		getUserCall.enqueue(object: Callback<User> {
			override fun onResponse(call: Call<User>, response: Response<User>) {
            continuation.resume(response.body())
		}

		override fun onFailure(call: Call<User>, t: Throwable) {
			continuation.resumeWithException(t)
		}
	}
```

Here we ar using a `suspendCoroutine {}` to convert a callback into a co-routine. In `getUser()` first we launch a `suspendCoroutine` block. 

	**INFO:** In order to use `suspendCoroutine{}` we declare `getUser()` as a `suspend` function.
	{: .notice--info}

It supplies a `continuation` as an parameter. We will be using this `continuation` to inform the coroutine to continue from the point where the function was suspended. We can also pass a result to the `continutaion` which will be returned when the suspended function returnes.
