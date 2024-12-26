---
title: "Find and Remove Files That Have Spaces In The Name"
date: "2010-07-09"
categories: 
  - "linux"
---

Whenever I need this, it takes me seemingly forever to figure this out. Blog it!

'find' is a great GNU utility for digging up files in directories. 'rm' is a staple command for any linux user. Combining the two commands can be fun and powerful.

```bash
find /home/luser -type f -name '\*.mpg' -exec rm -f {} \\;
```

The above command calls `rm -f` for each file. A faster version of this uses xargs for a single call.

```bash
find /home/luser -type f -name '\*.mpg' | xargs rm -f
```

Hooray! We just deleted all mpg files in luer's home directory. Unless, of course, any of those files had a name that included a space. Doh! One way to get around this is to use this modified version of the above command.

```bash
find /home/luser -type f -name '\*.mpg' | tr "\\n" "\\000" | xargs -0 rm -f
```

**Update:** As Stuart notes in the comments, you should be able to replace the 'tr' command with the find -print0 option. I wasn't able to get it to work, but maybe it helps you!
```bash
find /home/luser -type f -name '\*.mpg' -print0 | xargs -0 rm -f
```

**Update #2:** As Ole notes in the comments, GNU parallel is another option which provides parallel process running.
```bash
find /home/luser -type f -name ‘\*.mpg’ | parallel rm -f
```
