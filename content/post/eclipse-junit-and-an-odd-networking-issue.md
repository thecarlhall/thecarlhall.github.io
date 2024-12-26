---
title: "Eclipse, JUnit and an odd networking issue"
date: "2009-12-10"
categories: 
  - "development"
tags: 
  - "eclipse"
  - "ipv6"
  - "java"
  - "networking"
---

My coworker started having issues with getting JUnit to do anything in Eclipse. After some digging around, it was discovered that, for whatever reason, the JUnit runner was trying to create a network connection and couldn't. He tried to perform an update of Eclipse plugins and that failed also, for the same network connectivity reasons.

After a lot of digging and much gnashing of teeth, [he came across this page](http://pvaneynd.livejournal.com/132635.html) which outlines an issue where Java has started connecting using IPv6. We're not sure if this is a Debian package issue or something from elsewhere but thankfully the LiveJournal page outlines a couple of fixes. The fix performed at the system level rather than by Java properties is what fixed things for him.

To check the system property, do this:

```bash
machine:/# sysctl net.ipv6.bindv6only
```

Which, if you're having this problem should return 1. If you're having this problem and get a return value of 1, issue this command next:

```bash
machine:/# sysctl net.ipv6.bindv6only=0
```
