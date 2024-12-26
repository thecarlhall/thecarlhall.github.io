---
title: "Understanding the 'unresolved constraint', 'missing requirement' message from Apache Felix"
date: "2012-01-19"
categories: 
  - "development"
tags: 
  - "bundles"
  - "debugging"
  - "dependency-resolution"
  - "osgi"
---

It's pretty common while developing an OSGi bundle that your imports and exports won't quite match what you need or what exists in the server you're deploying to. This can show up as `NoClassDefFoundError`, `ClassNotFoundException` or as log output in a stacktrace from bundle resolution. Hall, Pauls, McCullough and Savage did a great job of covering NCDFE and CNFE in ["OSGi In Action"](http://www.manning.com/hall/ "OSGi In Action") (chapter 8), let's take a look at figuring out what the bundle resolution stacktrace is telling us. _(I make nothing from the sales of "OSGi In Action" and suggest it to anyone interested in OSGi.)_

<!--more-->

Just like learning to read the stacktrace from an exception in Java is key to debugging, so is true about the dependency resolution messages from an OSGi container. Below is the output from [Apache Felix](http://felix.apache.org/site/index.html "Apache Felix") when it encountered a missing dependency required by a bundle:

```
ERROR: Bundle org.sakaiproject.nakamura.webconsole.solr [124]: Error starting slinginstall:org.sakaiproject.nakamura.webconsole.solr-1.2-SNAPSHOT.jar (org.osgi.framework.BundleException: Unresolved constraint in bundle org.sakaiproject.nakamura.webconsole.solr [124]: Unable to resolve 124.0: missing requirement [124.0] package; (package=org.apache.solr.client.solrj) [caused by: Unable to resolve 84.0: missing requirement [84.0] package; (package=org.sakaiproject.nakamura.api.lite) [caused by: Unable to resolve 86.0: missing requirement [86.0] package; (&amp;(package=com.google.common.collect)(version&gt;=9.0.0)(!(version&gt;=10.0.0)))]])
org.osgi.framework.BundleException: Unresolved constraint in bundle org.sakaiproject.nakamura.webconsole.solr [124]: Unable to resolve 124.0: missing requirement [124.0] package; (package=org.apache.solr.client.solrj) [caused by: Unable to resolve 84.0: missing requirement [84.0] package; (package=org.sakaiproject.nakamura.api.lite) [caused by: Unable to resolve 86.0: missing requirement [86.0] package; (&amp;(package=com.google.common.collect)(version&gt;=9.0.0)(!(version&gt;=10.0.0)))]]
    at org.apache.felix.framework.Felix.resolveBundle(Felix.java:3443)
    at org.apache.felix.framework.Felix.startBundle(Felix.java:1727)
    at org.apache.felix.framework.Felix.setActiveStartLevel(Felix.java:1156)
    at org.apache.felix.framework.StartLevelImpl.run(StartLevelImpl.java:264)
    at java.lang.Thread.run(Thread.java:619)
```

What you have here is a stacktrace with a lengthy message. The important part of the stacktrace for us _is_ the message.

```
ERROR: Bundle org.sakaiproject.nakamura.webconsole.solr [124]: Error starting slinginstall:org.sakaiproject.nakamura.webconsole.solr-1.2-SNAPSHOT.jar (org.osgi.framework.BundleException: Unresolved constraint in bundle org.sakaiproject.nakamura.webconsole.solr [124]: Unable to resolve 124.0: missing requirement [124.0] package; (package=org.apache.solr.client.solrj) [caused by: Unable to resolve 84.0: missing requirement [84.0] package; (package=org.sakaiproject.nakamura.api.lite) [caused by: Unable to resolve 86.0: missing requirement [86.0] package; (&amp;(package=com.google.common.collect)(version&gt;=9.0.0)(!(version&gt;=10.0.0)))]])
```

This message is pretty simple but the structure is common for nastier messages (i.e. deeper resolution paths before failure). Let's pull it apart to see what's happening in there.

```
ERROR: Bundle org.sakaiproject.nakamura.webconsole.solr [124]: Error starting slinginstall:org.sakaiproject.nakamura.webconsole.solr-1.2-SNAPSHOT.jar
```

This very first part tells us that an error occurred while trying to load the `org.sakaiproject.nakamura.webconsole.solr`bundle. Nice start, but not quite the crux of the matter. Let's keep reading.

```
org.osgi.framework.BundleException: Unresolved constraint in bundle org.sakaiproject.nakamura.webconsole.solr [124]: Unable to resolve 124.0: missing requirement [124.0] package; (package=org.apache.solr.client.solrj) [caused by: Unable to resolve 84.0: missing requirement [84.0] package; (package=org.sakaiproject.nakamura.api.lite) [caused by: Unable to resolve 86.0: missing requirement [86.0] package; (&amp;(package=com.google.common.collect)(version&gt;=9.0.0)(!(version&gt;=10.0.0)))]])
```

Phew, that's a lot of text! This is the heart of what we need though, so let's break it down to make more sense of it.

```
(
    org.osgi.framework.BundleException: Unresolved constraint in bundle org.sakaiproject.nakamura.webconsole.solr [124]: Unable to resolve 124.0: missing requirement [124.0] package; (package=org.apache.solr.client.solrj)
        [
            caused by: Unable to resolve 84.0: missing requirement [84.0] package; (package=org.sakaiproject.nakamura.api.lite)
            [
                 caused by: Unable to resolve 86.0: missing requirement [86.0] package; (&amp;(package=com.google.common.collect)(version&gt;=9.0.0)(!(version&gt;=10.0.0)))
            ]
        ]
)
```

### What are those `[number]`s in the message?

The numbers in the message tell us the bundle ID on the server.

| Unresolved Package Name | Bundle ID Where Resolution Failed |
| --- | --- |
| org.apache.solr.client.solrj | 124 |
| org.sakaiproject.nakamura.api.lite | 84 |
| com.google.common.collect | 86 |

Once you pull apart the message it becomes more obvious that it has structure and meaning! The structure of the message tells us that bundle 124 depends on a package from bundle 84 which depends on a package from bundle 86 which is unable to resolve `com.google.common.collect;version=[9.0.0, 10.0.0)`. The innermost/very last message tells us the root of the problem; the dependency resolver was unable to find `com.google.common.collect` at `version=[9.0.0, 10.0.0)`. Now we have somewhere to start digging.

### How To Fix This

I suggest one of the following steps:

1. Add a bundle that exports the missing package with a version that matches the required version

3. Change the version to match an exported package already on the server

In this particular environment, `com.google.common.collect;version=10.0.0` is what our server has deployed. The descriptor above specifically blocks any version not in the 9.x.x range. We generate the OSGi manifest by using the [Maven Bundle Plugin](http://felix.apache.org/site/apache-felix-maven-bundle-plugin-bnd.html "Maven Bundle Plugin") which uses the BND tool to generate the manifest. In BND version > 2.1.0, the [macro for versions](http://www.aqute.biz/Bnd/Versioning "macro for versions") was changed. Our solution has ranged from rolling back to bnd version=2.1.0 **OR** [define the macro differently](http://davidvaleri.wordpress.com/2011/04/07/secrets-of-the-felix-bundle-plug-in-macros-revealed/ "define the macro differently"). The results are the same; the version segment in the manifest header becomes `com.google.common.collect;version>=9.0.0` which finds our bundle of `com.google.common.collect;version=10.0.0`.

### Notes about environment

The above message and stacktrace originated from a Sakai OAE environment which is built on Apache Sling and thusly Apache Felix. We use an artifact ID that is the root package of the bundle (org.sakaiproject.nakamura.webconsole.solr). This has the side effect that our bundle names look like package names in the message but gives a very clear naming convention.
