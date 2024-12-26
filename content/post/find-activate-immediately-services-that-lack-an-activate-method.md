---
title: "Find activate-immediately services that lack an activate method"
date: "2012-07-30"
categories: 
  - "development"
  - "osgi"
---

Immediate service activation in OSGi can be tricky but there are [some basic rules to consider](http://thecarlhall.wordpress.com/2011/02/04/when-to-immediately-activate-an-osgi-component/ "some basic rules to consider"). Another point to think about is when a service doesn't contain an activate method. The code base I work in uses Felix's SCR annotations which makes this search pretty concise. I also assume that the code is in git. If your code isn't, you should be able to replace `git` with `find <dir> -type f -exec <grep-fu here>`.

<!--more-->

## Find activate-immediately services

Using a little git and grep-fu, we can find our services that activate immediately.

`git grep -l 'immediate[[:space:]]\?=[[:space:]]\?true'`

The command below makes a few assumptions:

1. We use SCR annotations that allow `immediate = (true|false)`

3. We use `immediate = true` almost never except in the `@Component` annotation.

## Find services that have an activate method

For our next trick, we'll find the set of files that have an activate method as noted by the use of an `@Activate` annotation.

`git grep -l '@Activate'`

## Find the intersection of activate-immediately services that lack an activate method

Now to the fun. With the information we've grepped above, we need the things that are in set 1 but _not_ in set 2. There's a great little tool in linux called `comm` that can help us with the set operations. It needs sorted data to work correctly, so we'll dress up our previous commands and send them in.

`comm -23 &lt;(git grep -l 'immediate[[:space:]]\?=[[:space:]]\?true' | sort) &lt;(git grep -l '@Activate' | sort)`

The `-23` argument tells `comm` that we want to suppress unique items from the second set (activate method list) and to suppress items that exist in both sets (immediate + activate). This leaves us with services that don't have an activate method. If you want to see services that are immediate _and_ have an activate method, change the argument to `-12`.
