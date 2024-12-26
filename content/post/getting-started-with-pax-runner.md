---
title: "Getting started with Pax Runner"
date: "2012-07-11"
categories: 
  - "osgi"
---

After fighting through a [Maven assembly for a small project](https://source.sakaiproject.org/contrib//cans/cans_AA/trunk/dist/src/main/assembly/bin.xml "Maven assembly for a small project"), I just couldn't take that headache again. I've used [Apache Sling's Maven Launchpad Plugin](http://sling.apache.org/site/maven-launchpad-plugin.html "Apache Sling's Maven Launchpad Plugin") to put together a standalone OSGi server but Launchpad doesn't allow you to pick which OSGi container you deploy to or what version of Felix gets used. I've started working with [Pax Runner](http://team.ops4j.org/wiki/display/paxrunner/Pax+Runner "Pax Runner") because a) it looks pretty nifty, b) those Pax folks are doing great stuff for the OSGi deployers out there.

[Pax Runner](http://team.ops4j.org/wiki/display/paxrunner/Pax+Runner "Pax Runner") has a few sweet features I'm really digging right now.

<!--more-->

### Different OSGi platforms and versions

I generally develop on Apache Felix, but if I should be able to run in any other OSGi container, right? Well, to test that theory I can ask [Pax Runner](http://team.ops4j.org/wiki/display/paxrunner/Pax+Runner "Pax Runner") to load up my profile using a different platform (`--platform`; defaults to '`felix`') and the version of that platform (`--version`).

If I want to test my setup with Equinox I just run:

```
pax-run --platform=equinox awesome-profile.composite
```

Knopflerfish you say?

```
pax-run --platform=knopflerfish awesome-profile.composite
```

What about an older version of Felix? Easy!

```
pax-run --platform=felix --version=3.0.8 awesome-profile.composite
```

### Deploy multiple profiles and build composite profiles

I plan to run my project with a single profile, but if you find the need to include other profiles it's just another command line switch:

```
pax-run --profiles=config,log awesome-profile.composite
```

_"But I want a specific version of a profile."_ And you should! So use this:

```
pax-run --profiles=config/1.0.0,log/1.2.0 awesome-profile.composite
```

### Profiles? What's this crazy talk you speak?!

So, the root of all this chatter is Pax Runner Profiles. I can barely speak to the topic, but will attempt to anyway. (Don't trust me; [read the documentation](http://team.ops4j.org/wiki/display/paxrunner/Advanced+profiles+topics "read the documentation"))

A short glimpse into my profile file shows that I just include bundles that I know to live in a Maven repository:

```
scan-bundle:mvn:commons-io/commons-io/1.4@1
scan-bundle:mvn:commons-fileupload/commons-fileupload/1.2.2@1
scan-bundle:mvn:commons-collections/commons-collections/3.2.1@1
scan-bundle:mvn:commons-lang/commons-lang/2.6@1
scan-bundle:mvn:commons-pool/commons-pool/1.5.6@1
scan-bundle:mvn:commons-codec/commons-codec/1.5@1
```

A quick explanation of these lines is simply:  
`scan-bundle:mvn:<groupId>/<artifactId>/<version>@<startLevel>`

For those looking for more OSGi goodness, [Pax Web](http://team.ops4j.org/wiki/display/paxweb/Pax+Web "Pax Web") has really stepped up with the [2.0.0 release](http://team.ops4j.org/wiki/display/paxweb/Pax+Web+-+2.0.0 "2.0.0 release"). You can configure your Jetty server by deploying a bundle fragment with the appropriate `jetty.xml` file. Awesome!
