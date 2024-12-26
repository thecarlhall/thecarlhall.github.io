---
title: "Unit Testing with HttpClient's LocalTestServer"
date: "2010-03-25"
categories: 
  - "testing"
tags: 
  - "httpclient"
  - "java"
  - "unit-tests"
---

When unit testing code that uses HttpClient, it can get a bit tricky to not test against a active web server.  There are a couple of approaches to this to keep your tests at the unit level.

1. Mock and inject HttpClient -- While this is certainly possible and gives you complete control, it can take a lot of mocking and get tedious quite quickly.
2. Use LocalTestServer from HttpClient -- This is the point of this post and I will now explain.

Let's take a look at the basic setup for getting this test server up.

### Basic Setup

First, if you're using Maven, you'll need to bring in a new depedency.

```xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.0.1</version>
    <classifier>tests</classifier>
    <scope>test</scope>
</dependency>
```

Next, you'll need to setup your unit test with the test server.

```java
public void setUp() {
    LocalTestServer server = new LocalTestServer(null, null);
    server.start();
}
```

That's all it takes to get the server in place and started but it's not very useful without some handlers.

### Adding Handlers

To really get something worthwhile out of the server, you'll want to register at least one handler.  Your handler must implement

`org.apache.http.protocol.HttpRequestHandler`

which has one really obvious method,

`void handle(HttpRequest request, HttpResponse response, HttpContext context) throws HttpException, IOException;`

Since we should be in a unit test, you can decide to either Mock this interface or create concrete classes that implement it.

Now that we have this spiffy way of controlling the response of the server, let's see what it takes to register this handler with the server.

```java
// do something to mock this or instantiate your own concrete class HttpRequestHandler handler;

server.register("/someUrl/\*", handler);
```

That's all it takes to setup a local test server with at least 1 registered handler listening on /someUrl. For clarity, let's take a look at the code all together.

```java
public class MyUnitTest {
    private LocalTestServer server = null;

    @Mock HttpRequestHandler handler;

    @Before public void setUp() {
        server = new LocalTestServer(null, null);
        server.register("/someUrl/\*", handler);
        server.start();

        // report how to access the server String serverUrl = "http://" + server.getServiceHostName() + ":" + server.getServicePort();
        System.out.println("LocalTestServer available at " + serverUrl);
    }

    // do lots of testing!

    @After public void tearDown() {
        server.stop();
    }
}
```
