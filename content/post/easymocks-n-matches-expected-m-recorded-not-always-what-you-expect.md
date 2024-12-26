---
title: "EasyMock's \"N matches expected, M recorded\" not always what you expect"
date: "2009-12-03"
categories: 
  - "testing"
tags: 
  - "easymock"
  - "java"
---

This has been discussed in varying ways. I tend to point to [this article](http://www.springone2gx.com/blog/scott_leberknight/2008/09/the_n_matchers_expected_m_recorded_problem_in_easymock) as a good description of how to understand what is going on when this message is given. There is an interesting corner case that I feel needs to be explored as I've just wasted an hour trying to figure it out.

Consider the following code sample.

```java
public void testSomethingBasic() {
    Thing myThing = new Thing(EasyMock.isA(Whatever.class));
    // do more stuff in the test
}
public void testSomethingElse() {
    IJobber myJobber = EasyMock.createMock(IJobber.class);
    EasyMock.expect(myJobber.doStuff()).andReturn(null);
    // do more stuff in the test
}
```

The thing to notice is that _isA(..)_ was used on a non-mock object then used again on a mock object. The problem here is that EasyMock will record that you created a matcher (ie. isA(Whatever.class)) that was never used then will record that you created another one that you did use. You'll get an error to the nature of "1 matcher expected, 2 recorded".

While this is not always the case, a debugging method to consider is when you have more recorded than expected, check test methods that run before the one throwing this exception to make sure you aren't improperly creating matchers that don't get used.
