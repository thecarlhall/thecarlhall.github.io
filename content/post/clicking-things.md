---
title: "Clicking Things"
date: "2009-12-02"
categories: 
  - "linux"
tags: 
  - "eclipse"
  - "flash"
  - "gtk"
---

For a month or so now, I've been having problems with clicking buttons in [Eclipse](http://www.eclipse.org/) and in [Flash](http://www.adobe.com/products/flash/). After asking in the Eclipse IRC room, I was able to find an answer that actually fixes both Eclipse and Flash.

**Eclipse** (from [the wiki](http://wiki.eclipse.org/IRC_FAQ#Eclipse_buttons_in_dialogs_and_other_places_are_not_working_for_me_if_I_click_them_with_the_mouse._They_work_if_I_use_the_mnemonic_key_though.2C_what.27s_going_on.3F)) I've created a startup script that sets the following system property. `export GDK_NATIVE_WINDOWS=true`

**Flash** (from [this post](http://ubuntuforums.org/showthread.php?p=8409479)) Pretty much the same fix as above but made in a different place. In _/usr/lib/nspluginwrapper/i386/linux/npviewer_, add the following: `export GDK_NATIVE_WINDOWS=1`

_before_ the line: `. /usr/lib/nspluginwrapper/noarch/npviewer` A note to all this. Don't put either of these properties in .profile, .bashrc or anything like that. This property needs to be set specific to things that break as this changes how GTK+ processes events.
