---
title: "JavaMail in OSGi"
date: "2009-10-27"
categories: 
  - "development"
tags: 
  - "java"
  - "javamail"
  - "osgi"
---

## UnsupportedDataTypeException

Sun decided ages ago that JavaMail and the Java Activation Framework should be released as separate artifacts and no one really cared. If you need JavaMail, you knew to also include JAF as Sun's JavaMail page also told what version of JAF to use. Using this same approach, I created and deployed separate bundles for javax.activation and javax.mail. As soon as I tried to send the first test email, I found trouble. The most basic of email content types, text/plain, could not be sent.

```
javax.activation.UnsupportedDataTypeException: no object DCH for MIME type text/plain at javax.activation.ObjectDataContentHandler.writeTo(DataHandler.java:885)
```

This sent me digging into the source of javax.activation and javax.mail to find the fix. The problem here is that when the Java Activation Framework starts up, it loads META-INF/mailcap.defaults then looks for META-INF/mailcap files in other libraries in the same classloader. These mailcap files define handlers for specific types of content (eg. text/plain). Since JAF and JavaMail were in separate bundles and thusly classloaders, the mailcap file in JavaMail couldn't be seen by JAF. The fix I went with was to merge the 2 bundles into 1. I didn't want to do this at first but after I saw that JAF uses an internal class named MailcapCommandMap, I decided that if the Sun developers felt it fine to give JAF explicit knowledge of JavaMail, it was fine with me to marry them to the same bundle. If you're using Equinox, you can use buddy classloaders without merging the bundles together, but I have a couple of problems with buddy classloaders.

1. They are not part of the OSGi spec and break your ability to move to another container.
2. They create a situation where a parent bundle has to have knowledge of children or "buddy" bundles. This goes against the basic whiteboard approach of OSGi.

## Unable to resolve sun.security.util

Another fun bit to note is that starting with JavaMail 1.4.1, there is use of the sun.security.util package. If your OSGi container exports sun.\* classes, you won't notice this. If your OSGi container doesn't export this or you run on a non-Sun JVM, this seems like a blocking issue. Fear not! After digging through the JavaMail code I find a comment in the SocketFetcher class that reads:

```java
/*
 * First, try to use sun.security.util.HostnameChecker,
 * which exists in Sun's JDK starting with 1.4.1.
 * We use reflection to access it in case it's not available
 * in the JDK we're running on.
 */
```

This let me know that I can set this package to optional in my import list and all would be well. In other words, your bundle manifest should have something like this:

`Import-Package: sun.security.util;resolution:=optional`
