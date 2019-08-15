---
title: How I renamed an entire library already in production
categories:
  - Android
tags:
  - Android
  - Android-Studio
  - Bintray
  - Github
  - Jcenter
  - Library
---

In an attempt to learn more about Kotlin and to also learn more about publishing and distribution of libraries, I built a library that lets you drag to give rating which also allows you to customise assets to the rating.

![](/assets/images/demo.gif){: .align-center}

For some reason initially I though SlideRating was the perfect name for the library and it was published with that name on Github and bintray as well. (Yes!! I still can’t believe I decided to go ahead with that) Then I felt this name doesn’t do justice to the library and what it provides, so I decided to rename it to DragRating.

## Changing the name of library

To change the name of an already published library, I broadly listed down the place where the name will have to be update.

1. Rename the project (in Android Studio)
1. Rename the modules, classes and package name in the codebase. (if any)
1. Change the name of the repo on Github.
1. Hosting the library with the new name on bintray and jcenter.

### 1. Rename the name of the project (in Android Studio)
This can be done very easily by following the below steps.

1. Close your Android Studio project.
1. Rename the root project directory name from your File Manager (I renamed the project from `SlideRating` to `DragRating`
1. Open Android Studio
1. Open the new project by browsing to it on Android Studio
1. Clean build project (This will create all the new files required by the project.)
1. Add the line `rootProject.name = 'DragRating'` to the `settings.gradle` file of the project.

### 2. Rename the modules, classes and package name in the codebase (if any)
This is a completely optional step and actually depends based on your need. In my case, I had to change the name of the library module, package name and some resource values. Which I did accrodingly.

1. Rename the module (Refactor -> Rename on module)
1. Refactor the package (Refactor -> Rename -> Rename Package)
1. Rename the classes names in the package (Refactor -> Rename)
1. Rename other files such as values resources and update the comments with the new class names which were not updated (Edit -> Find -> Find in path…) and manually rename those values.

### 3. Change the name of the repo on Github
This is the easiest of all the steps. We just have to rename the repo and update the new URL of the origin, locally.

1. Rename the Github repo by opening the project -> Settings -> “Repository name”
1. Update the remote git repo URL for our local repository. For this run the command `git remote set-url origin <<new .git url>> <<old .git url>>` in it, replace the `<<new .git url>>` and `<<old .git url>>` like this `git remote set-url origin git@github.com:saran2020/DragRating.git git@github.com:saran2020/SlideRating.git`

### 4. Hosing the library with the new name on Bintray and jcenter
1. Create a new repo with the new name on Bintray. Editing an existing repo is not allowed
1. Create a new package with the name you want for the library to be included in the project with.
1. Update the Android project to point towards the new repo and make the necessary changes that have to be done, before a new version is sent out.
1. Upload the new version to bintray, as you would normally do
1. Send a request to publish the new repo to jcenter. This step can only be done after step 4 because bintray doesn’t allow you to send a request to jcenter for an empty project.
1. Wait for the repo to be approved by jcenter to be allowed to be published. After it’s approved, you will receive an E-mail. It usually takes a day’s time for the package to be approved.
1. Congratulations! Your library with the new name has been published successfully.

### 5. PRO TIPS

1. Always use Refactor to save all your headache. It will always shows all the changes that will be made for that refactor. It’s a good idea to go through them then, or before committing the changes to git.
1. It’s a good strategy to keep running the project after every step or after you have made substantial progress.
> Keep committing after every successful run.
1. Don’t delete the previous repo, because then old users won’t be able to access them. The people who have not updated the dependency will still need the older version.
1. It’s a good idea to add the change of name to the Readme for the old users coming back to know.

It only took a couple of hours for me, since the project I was migrating was not very big.
