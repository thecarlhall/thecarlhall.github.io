---
title: "Git rid of a remote branch"
date: "2009-04-05"
categories: 
  - "source-control"
tags: 
  - "git"
---

I keep having to look this up so I figured I'd blog it to save the Google search.

I sometimes clone repos from another git repo and from svn. This allows me to publish to both and do whatever I want locally. The default prefix for the first cloned git repo is "origin". As I sync more repos to my local repo, I like to rename the prefix to something more indicative of the branches origin.

To rename the prefix "origin" to "moe", start with something like this:

> \[remote "origin"\] url = moe:src/howler fetch = +refs/heads/\*:refs/remotes/origin/\*

and change it to this:

> \[remote "moe"\] url = moe:src/howler fetch = +refs/heads/\*:refs/remotes/moe/\*

After this, call "git fetch moe" to fetch the list of remote branches. The same approach can be taken to rename branches fetched from svn. The important thing to note is to look for the name to change after "remotes" in the config line. Adding something before "remotes" will do nothing for you.

The problem you're left with, however, is that if you fetched the remote branches before and after the change, you'll now have some branches that need pruning. To clean up all the now useless branch names, use a command like this:

> git branch -a | grep '^ origin' | xargs git branch -rd

This should remove any remote branches that begin with 'origin'. If you delete the wrong remote branches, you can always get them back by fetching again.
