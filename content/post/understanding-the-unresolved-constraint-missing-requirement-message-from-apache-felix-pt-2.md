---
title: "Understanding the ‘unresolved constraint’, ‘missing requirement’ message from Apache Felix Pt. 2"
date: "2012-07-12"
categories: 
  - "osgi"
---

We previously took a look at [Felix's unresolved constraint message](http://thecarlhall.wordpress.com/2012/01/19/understanding-the-unresolved-constraint-missing-resource-message-from-apache-felix/ "Felix's unresolved contract message"). I started testing with Felix 4.0.2 today and realized the output for an unresolved constraint has changed a bit.

```
ERROR: Bundle org.sakaiproject.nakamura.world [76] Error starting file:bundles/org.sakaiproject.nakamura.world_1.4.0.SNAPSHOT.jar (org.osgi.framework.BundleException: Unresolved constraint in bundle org.sakaiproject.nakamura.world [76]: Unable to resolve 76.0: missing requirement [76.0] osgi.wiring.package; (&(osgi.wiring.package=javax.servlet)(version>=3.0.0)))
11.07.2012 17:01:43.297 *ERROR* [FelixDispatchQueue] org.sakaiproject.nakamura.world FrameworkEvent ERROR (org.osgi.framework.BundleException: Unresolved constraint in bundle org.sakaiproject.nakamura.world [76]: Unable to resolve 76.0: missing requirement [76.0] osgi.wiring.package; (&(osgi.wiring.package=javax.servlet)(version>=3.0.0))) org.osgi.framework.BundleException: Unresolved constraint in bundle org.sakaiproject.nakamura.world [76]: Unable to resolve 76.0: missing requirement [76.0] osgi.wiring.package; (&(osgi.wiring.package=javax.servlet)(version>=3.0.0))
 at org.apache.felix.framework.Felix.resolveBundleRevision(Felix.java:3826)
 at org.apache.felix.framework.Felix.startBundle(Felix.java:1868)
 at org.apache.felix.framework.Felix.setActiveStartLevel(Felix.java:1191)
 at org.apache.felix.framework.FrameworkStartLevelImpl.run(FrameworkStartLevelImpl.java:295)
 at java.lang.Thread.run(Thread.java:679)
```

We have the same unresolved constraint..missing requirement as before but the part we're interested in has changed a bit. Let's break apart that first message.

<!--more-->

```
(org.osgi.framework.BundleException: Unresolved constraint in bundle org.sakaiproject.nakamura.world [76]: Unable to resolve 76.0: missing requirement [76.0] osgi.wiring.package; (&(osgi.wiring.package=javax.servlet)(version>=3.0.0)))
```

Given what we know from last time, the interesting bits above are:

```
Unresolved constraint in bundle org.sakaiproject.nakamura.world [76]
```

This tells you what bundle had an issue trying to resolve a constraint. The next part is a bit less obvious but can be broken up.

```
osgi.wiring.package; (&(osgi.wiring.package=javax.servlet)(version>=3.0.0)))
```

osgi.wiring.package looks pretty foreign, hu? Disregard that and you see `javax.servlet` and `version>=3.0.0`. Tada! That's the good stuff. So, figure out where that bundle is that exports `javax.servlet>=3.0.0` and you're on your way. (Hint: maybe, `javax.servlet:javax.servlet-api:3.0.0` or `org.ops4j.pax.web:pax-web-jetty-bundle:2.0.1` is what you're looking for.)
