---
title: "Getting Thrift into Maven Locally"
date: "2010-08-27"
categories: 
  - "development"
tags: 
  - "java"
  - "java-ant-maven"
---

I spent the better part of a morning trying to figure out how to get Thrift to be available in a Maven repository for a project I'm working on. Yes, I know there's [a JIRA issue for this](https://issues.apache.org/jira/browse/THRIFT-363) but the comments are not clear and it's more tailored towards publishing in the central Apache repository. Also, I tried using 'ant publish' and got "Execute failed: java.io.IOException: Cannot run program "../../compiler/cpp/thrift": java.io.IOException: error=2, No such file or directory". I don't need that kind of problem installing a library nor do I want to build the whole Thrift library.

What I did to finally get this into my local maven repository is this.

1. Download and unarchive the latest stable Thrift release
    - [http://incubator.apache.org/thrift/download/](http://incubator.apache.org/thrift/download/)
2. Change into the Thrift Java root directory
    - cd lib/java
3. Run the default ant target, "dist"
    - ant
4. Install the build artifact into the local Maven repository. _Note: Be sure to update the version to match what you downloaded._

- mvn install:install-file -Dfile=libthrift.jar -DgroupId=org.apache.thrift -DartifactId=libthrift -Dversion=0.4.0 -Dpackaging=jar -DgeneratePom=true
