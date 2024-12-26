---
title: "When To Immediately Activate An OSGi Component"
date: "2011-02-04"
categories: 
  - "development"
tags: 
  - "osgi"
  - "services"
---

OSGi has a fantastic feature for immediate components [\[1\]](#1_osgi) and delayed components [\[2\]](#2_osgi). This allows components to delay their possibly expensive activation until the component is first accessed. At the very least it allows the OSGi platform to consume resources as needed. No sense is sucking up those server resources for something that can wait.

As developers start their adventure into OSGi, I've noticed a pattern creeping up that I'm not sure folks are aware of: j_ust because you like your component doesn't mean it needs to start immediately._ Unless your component has something it needs to do before other components can do their work, there's little need to start a component immediately. A lot of times, the work can be moved into the bundle activator or is fine being delayed.

People often see that a component does _something_ in its activation method and assume it should start immediately. The questions to ask yourself are:

- Will the things that need to use this component have a reference to it? _Yes? Delayed activation._
- Before activation, will functionality be missing that should be available as soon as possible? _Yes? Immediate activation._
- Can something else do the job of activating the component at a later time? _Yes? Delayed activation._

Registering with an external service, such as JMS, is a perfect scenario for an immediate component. If the JMS service were in OSGi, declarative services could do the work of making sure service trackers know your component exists, but OSGi has no jurisdiction over external resources. Binding to a JMS topic requires manual interaction and therefore should happen by way of immediate component activation.

Getting and setting of properties is not a case for immediate component activation. Neither is because the service is referenced by something else.

When using declarative services, a reference to your not-yet-started component will be given to anything that has said it wants to collect an interface you implement. This requires no effort on the part of the service being referenced and thusly does not require the referenced service to be activated immediately. Fret not! On first access of this service by any consumer the service will get activated.

OSGi has not been the fastest thing for me to learn but understanding the nuances of little things like this really helps me understand why it is a good approach at decoupled services and modular software construction.

1\. OSGi compendium spec version 4.2, section 112.2.2 2\. OSGi compendium spec version 4.2, section 112.2.3
