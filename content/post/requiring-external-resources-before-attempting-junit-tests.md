---
title: "Requiring External Resources Before Attempting JUnit Tests"
date: "2015-11-15"
categories: 
  - "development"
tags: 
  - "dynamodb"
  - "integration-tests"
  - "junit"
---

If you have an integration test that requires external resources to be available, like [a local DynamoDB server](https://thecarlhall.wordpress.com/2015/11/14/integration-testing-with-dynamodb-locally/), that test should be skipped rather than fail when the resources aren't there. In JUnit, this can be accomplished by throwing an `AssumptionViolatedException` from an `@BeforeClass` method, or better yet, with reusable `ClassRule`s.

<!--more-->

A `ClassRule` runs like an `@BeforeClass` method; once before the entire suite of test methods in the class. `@Rule` is analogous to `@Before` and runs before each test method. In this case, we use the `ClassRule` to check for resources along with preparing the use of them (see the call to `.client()`). If the AWS client cannot connect to a local DynamoDB server, it will throw an `AssumptionViolatedException` causing JUnit to mark the test class as `skipped` rather than `failed`.

```
-------------------------------------------------------
 T E S T S
 -------------------------------------------------------
 Running DynamoDaoIT
 Tests run: 1, Failures: 0, Errors: 0, Skipped: 1, Time elapsed: 0.054 sec - in DynamoDaoIT

 Results :

 Tests run: 1, Failures: 0, Errors: 0, Skipped: 1
```

```java
import java.util.Properties;

import org.junit.Before;
import org.junit.ClassRule;
import org.junit.Test;

public class DynamoMappingsDaoIT {
  private static final Properties CONFIG = // load your app test properties;

  /**
   * This runs like an @BeforeClass method (@Rule is analogous to @Before).  If the AWS
   * client cannot connect to a local DynamoDB server, it will throw an
   * AssumptionViolatedException causing JUnit to mark the test class as &lt;code&gt;skipped&lt;/code&gt;
   * rather than &lt;code&gt;failed&lt;/code&gt;.
   */
  @ClassRule
  public static final DynamoDbRule DYNAMO_DB_RULE = new DynamoDbRule().withLocalPort(CONFIG.getString(&quot;dynamo.port&quot;));

  private DynamoDAO dao;

  @Before
  public void setUp() {
    // Create this here to give the class rule a chance to run first.
    dao = new DynamoDAO(CONFIG);
  }

  @Test
  public void shouldUseDynamo() {
    // test safely knowing that your dynamo client is connected
    mappingDAO.find(1);
    assertNotNull(DYNAMO_DB_RULE.client());
  }
}
```

```java
import com.amazonaws.AmazonClientException;
import com.amazonaws.ClientConfiguration;
import com.amazonaws.auth.AWSCredentials;
import com.amazonaws.auth.BasicAWSCredentials;
import com.amazonaws.services.dynamodbv2.AmazonDynamoDBClient;
import com.google.common.base.Strings;
import org.junit.Assume;
import org.junit.rules.ExternalResource;

public class DynamoDbRule extends ExternalResource {
  private String endpoint = &quot;http://localhost:8000&quot;;
  private String accessKeyId;
  private String secretKey;
  private int maxRetry = 0;
  private AmazonDynamoDBClient client;

  public DynamoDbRule() { }

  public DynamoDbRule withLocalPort(final int port) {
    return withEndpoint(&quot;http://localhost:&quot; + port);
  }

  public DynamoDbRule withEndpoint(final String endpoint) {
    this.endpoint = endpoint;
    return this;
  }

  public DynamoDbRule withAccessKeyId(final String accessKeyId) {
    this.accessKeyId = accessKeyId;
    return this;
  }

  public DynamoDbRule withSecretKey(final String secretKey) {
    this.secretKey = secretKey;
    return this;
  }

  public DynamoDbRule withMaxRetry(final int maxRetry) {
    this.maxRetry = maxRetry;
    return this;
  }

  public AmazonDynamoDBClient client() {
    if (client == null) {
      throw new IllegalStateException(&quot;Run before() to create the client&quot;);
    }
    return client;
  }

  @Override
  protected void before() throws Throwable {
    ClientConfiguration clientConfig = new ClientConfiguration().withMaxErrorRetry(maxRetry);

    if (Strings.isNullOrEmpty(accessKeyId) || Strings.isNullOrEmpty(secretKey)) {
      client = new AmazonDynamoDBClient(clientConfig);
    } else {
      AWSCredentials creds = new BasicAWSCredentials(accessKeyId, secretKey);
      client = new AmazonDynamoDBClient(creds, clientConfig);
    }
    client.setEndpoint(endpoint);

    try {
      client.listTables(1);
    } catch (AmazonClientException e) {
      Assume.assumeNoException(e);
    }
  }
}
```
