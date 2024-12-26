---
title: "IntelliJ IDEA shortcut in linux"
date: "2010-04-10"
categories: 
  - "development"
  - "linux"
tags: 
  - "gnome"
  - "intellij"
---

After a lot of digging around, I finally came across [this blog entry](http://www.cs.bgu.ac.il/~gwiener/programming/how-to-make-intellij-idea-8-usable-on-linux/) telling how to make a shortcut to IDEA work in linux. In short, change your shortcut to something like the following:

```bash
/bin/sh -c "export JDK_HOME=/usr/lib/jvm/java-6-sun && /opt/IntelliJIdea8/bin/idea.sh"
```

Voila! Victory is mine!
