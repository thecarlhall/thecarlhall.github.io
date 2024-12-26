---
title: "Integration Testing with DynamoDB Locally"
date: "2015-11-14"
categories: 
  - "development"
  - "testing"
tags: 
  - "dynamodb"
  - "integration-tests"
  - "maven"
---

One of the really nice things about using DynamoDB to back an application is the ability to write integration tests that have a good test server without trying to mimic DynamoDB yourself. [DynamoDB_Local is available from AWS](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Tools.DynamoDBLocal.html#Tools.DynamoDBLocal.DownloadingAndRunning) and is easily incorporated into a Maven build. Take a look through the documentation for [running DynamoDB on Your Computer](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Tools.DynamoDBLocal.html) for the parameters available.

<!--more-->

The general steps for adding this to your build are:

1. Download the DynamoDB Local artifact.

3. Reserve a local port for Dynamo to start on.

5. Start DynamoDB_Local just before integration tests are run.

7. Use the failsafe plugin to run integration tests.

The steps are covered in more technical detail in the pom.xml file below. Adding this to the build is the only thing necessary as this performs all the steps above and tears down the process afterwards.  
{{< gist thecarlhall f4e8f425cb736938e1d2 >}}
