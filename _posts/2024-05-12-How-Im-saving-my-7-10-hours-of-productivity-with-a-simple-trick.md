---
title: Productivity booster
categories:
  - Software engineering
tags:
  - git
  - project
  - build
---

In my role as a software engineer, it's crucial to maintain high levels of productivity. While some loss of productivity is due to ineffective habits like checking at our phone very often, some loss in productivity also happens due to ineffective use of technologies available to us. When working on Android a lot of my productivity is lost due to the extremely long & high memory-consuming Android builds. This is usually not a problem when I'm working on a feature where I have already run the build a few times, as most of the build output would be cached and only new changes get compiled. 

When a QA engineer raises a bug with small inconsistencies, such as missing alignment or a crash due to an [off-by-1 error](https://en.wikipedia.org/wiki/Off-by-one_error), it means I have to stop what I'm doing and address the reported issue. I dislike nothing more than having to switch branches, pull the master, fix the minor bug, run the build, and wait for the entire build to verify the fix. This becomes more frustrating when there's no build-cache because we are working on a new branch. Once the issue is fixed, I have to switch back to the previous branch, run the build again, and wait for the entire build to complete, all because this is a new branch.

When working in a team, we often learn something new from each other. Similarly, when working with a senior engineer, I learned a solution to this problem, which is to keep multiple copies of the repository.

Having multiple copies of the repository saves me from waiting for two fresh builds, which can cause 20-30 minutes of lost productivity. With multiple copies of the repository, I can verify the fix for the issue on my primary project along with the changes I was working on for a new feature. Once verified, I apply the same code in the secondary repository and publish it to raise the pull request.

I even go 1 step ahead and never open the secondary project in my IDE. Instead, I copy the file with the fix from the primary repository into the secondary repository. Of course, this only works for simple fixes.

I wish I had known this hack earlier in my career I would have saved hundreds of hours wasted switching the branch and triggering fresh builds.