---
title: "JCR Properties and Mixins"
date: "2009-03-16"
categories: 
  - "development"
tags: 
  - "jackrabbit"
  - "java"
  - "jcr"
  - "mixins"
  - "properties"
---

The more I learn about [JCR](http://en.wikipedia.org/wiki/Content_repository_API_for_Java) the more I realize I need to learn about JCR. Today's lesson: adding any property to an nt:file node.

In k2, we use nt:file as the node type for most everything we store especially anything stored by the user. There comes a time though when the node needs to have more characteristics than nt:file implies. This is where mixins come in handy. For more on mixins in general, read the 1st and 4th paragraphs of the [wikipedia article](http://en.wikipedia.org/wiki/Mixins).

We already have a mixin defined that allows a value be set to any property of a node (sakaijcr:properties-mix). Something to note here. I said a value to be set to any property. A. Why is this so important that I had to use 3 sentences to make my point? Because I spent several days trying to figure out why I couldn't save to a property I wanted to create even though the appropriate mixin had been added to the node.

A basic definition for allowing a value to be assigned to any property on a node looks like this: 
```xml
<propertyDefinition name="\*" requiredType="undefined" autoCreated="false" mandatory="false" onParentVersion="COPY" protected="false" multiple="false" />
```

See that multiple attribute there on the end? That tells if you will allow multiple values to be assigned to a property. This obviously does not allow multiple properties per the tell-tale "false" value.

More interestingly with this piece of configuration is that changing multiple to "true" disallows single values to be stored. I need to store either. What to do in such a quandary? Why, create another property definition, of course! Don't read it too fast or you may miss the subtle differences.
```xml
<propertyDefinition name="\*" requiredType="undefined" autoCreated="false" mandatory="false" onParentVersion="COPY" protected="false" multiple="false" />

<propertyDefinition name="\*" requiredType="undefined" autoCreated="false" mandatory="false" onParentVersion="COPY" protected="false" multiple="true" />
```

That's right. I changed the value of one attribute. This is how you can allow a property to have either a single value or multiple values. I feel a little dirty for duplicating so much for such a little change but the architects of JSR-170 felt it necessary to do so.

For those keeping count at home, I duplicated 153 characters between the 2 lines with the first having 1 more character than the second (158 vs. 157). If anyone knows a better way, I'm all ears.
