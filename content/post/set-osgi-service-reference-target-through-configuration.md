---
title: "Set OSGi Service Reference Target through Configuration"
date: "2010-06-06"
categories: 
  - "development"
tags: 
  - "java"
  - "osgi"
---

Passing properties into a component or component factory is pretty integral to OSGi.

A property can be a scalar or list of values.

```java
@Component(configurationFactory = true, policy = ConfigurationPolicy.REQUIRE) public class GreatExample {
    private static final String DEFAULT_VALUE = "someVal";
    @Property(value = DEFAULT_VALUE) static final String PROP_SOME_VAL = "prop.some.val";
    private String someVal;

    @Property static final String PROP_SOME_LIST = "prop.some.list";
    private List<String> someList;

    @Reference GreatService greatService;
    
    // later in the activate method
    protected void activate(Map<?, ?> props) {
        // do work to get properties and setting locally if needed.
        // For a good reference of how to properly retrieve properties
        // take a look at
        // http://svn.apache.org/viewvc/sling/trunk/bundles/commons/osgi/src/main/java/org/apache/sling/commons/osgi/OsgiUtil.java?view=markup
    }
}
```

The above example is terribly terse but the idea is pretty obvious. Now, from this, a more complex case can be built.

Let's say you have different implementations of GreatService and you need to configure the above class to use a certain one. This is usually done with the 'target' property using ldap query syntax. This is what we'll use to specify the wanted reference without hardwiring the particular implementation to our service consumer.

```java
@Component @Service @Property(name="type", value="really") public class ReallyGreatService implements GreatService { }
```

Using this arbitrary property, we are able to give a 'type' to this implementation. We can pick a particular implementation using this in conjunction with the reference target.

```java
@Reference(target="(type=really)") GreatService greatService;
```

While this is handy it is a compile time solution. The more flexible way to go about this is to make this a configuration time concern. Using the properties passed in when configuring the new GreatExample instance, we can tell the service reference what target to filter on.

```java
protected void activate(BundleContext bc) {
    // registering ReallyGreatService manually to make the example clearer.
    // could also be registered using SCR as the above annotations setup.
    GreatService gs = new ReallyGreatService();
    Hashtable gsProps = new Hashtable();
    gsProps.put("type", "really");
    ServiceRegistration reg = bc.registerService(GreatService.class.getName(), gs, gsProps);

    GreatExample ge = new GreatExample();
    Hashtable props = new Hashtable();
    props.put("greatService.target", "(type=really)");
    // desired implementation of GreatService ServiceRegistration reg = bc.registerService(GreatExample.class.getName(), ge, props);
}
```

If you have a specific service you want to bind to (I don't question your reasons; just giving examples), you can use the service.pid property of the service to set the target.
```java
protected void activate(BundleContext bc) {
    GreatExample ge = new GreatExample();
    Hashtable props = new Hashtable();
    props.put("greatService.target", "(service.pid=com.myCompany.SomeGreatServicePid)");
    // desired implementation of GreatService ServiceRegistration reg = bc.registerService(GreatExample.class.getName(), ge, props);
}
```

_Update (2011-01-10): Added more details to example and corrected class references._
