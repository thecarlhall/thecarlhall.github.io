---
title: "Setting Up RVM, Ruby, Gem and Rails on Ubuntu"
date: "2010-10-19"
categories: 
  - "development"
---

**This is centered on Ubuntu only because aptitude is used once. If you're comfortable with your package manager feel free to replace the aptitude command with your local command.**

I'm beginning to dig deeper into ruby, rails and the whole suite of tools available for them and have hit my knee on several things along the way. I'm no stranger to downloading things to download things to do things; I've been doing that in Java since 1996. I've also been kicking the tires on Ubuntu sense back before they got their naming scheme together and Debian even before that. I totally dig the Debian/Ubuntu way of install by deb packages. But with ruby, like Java, I feel it is best setup a machine by minimizing the use of apt (apt-get, aptitude, etc) and downloading the tools you need directly. I don't let aptitude install Eclipse, Xalan, servlet-api or any of the other tools/libraries I use on top of the JVM and this is the route I have ended up going with when setting up ruby as well.

## TL;DR: short list of what to do

This short list is for Ruby 1.9.2. If you want to install an earlier version, be sure to follow the extra steps about [installing gem from rubygems.org](#install_gem).

1. Install RVM [\[details\]](#install_rvm):
```bash
bash < > ~/.bashrc # This loads RVM into a shell session. source ~/.bashrc
```
2. Install a C compiler for RVM [\[details\]](#install_gcc): 
```bash
sudo aptitude install gcc
```
3. Install Ruby 1.9.2 using RVM [\[details\]](#install_ruby):
    
    _[See below for instructions on installing RubyGems when using Ruby < v1.9.2](#install_gem)_
    ```ruby
    rvm pkg install zlib rvm install 1.9.2 --default -C --with-zlib-dir=$HOME/.rvm/usr
    ```
4. Install Rails using RubyGems [\[details\]](#install_rails):
    
    ```bash
    gem install rails
    ```

## Details:

#### Install RVM

_Note: The [full install instructions](http://rvm.beginrescueend.com/rvm/install/) have much better details._

[RVM (Ruby Version Manager)](http://rvm.beginrescueend.com/) gives us a way to install Ruby without getting it from aptitude. Being that Ruby is our starting point for everything after RVM and RVM is our means of getting Ruby, everything is downhill after getting RVM. You could install ruby using aptitude then install rvm as a gem but what's the point of downloading Ruby just to download stuff to eventually download Ruby (again) and remove the system local stuff that was installed the first time. Let's just cut out all the cruft in the middle.

#### Install a C compiler for RVM

Since RVM setups up the Ruby interpreter, it has to act 1 level deeper than the interpreter and C is that next level down. GCC is a very common tool to have on a linux machine, so this step is mostly for setting up a new machine. It's always a good idea to have GCC installed. And vi. But this is a tech blog and not for theology, so I won't go into that.

 

#### Install Ruby 1.9.2 using RVM

The big moment! All this work to get Ruby installed so we can begin our journey into RoR-land. This will take some time to perform as it needs to download, configure and compile Ruby.

 

##### _Install RubyGems from rubygems.org for Ruby < v1.9.2_

_Note: The [full install instructions](http://docs.rubygems.org/read/chapter/3) have much better details._

```bash
wget http://production.cf.rubygems.org/rubygems/rubygems-1.3.7.tgz tar zxf rubygems-1.3.7.tgz cd rubygems-1.3.7 ruby setup.rb
```

Ruby 1.9.2 includes RubyGems 1.3.7, so we don't have to install it when we install Ruby, but for previous versions we'll need to install this bit ourselves. You can install it from aptitude but it locks you from doing system updates and some other functionality I wasn't keen on losting. Installing it from rubygems.org gives back the freedom to break my system in whatever way I choose.

#### Install Rails using RubyGems

This is where things get down to Ruby's style of easy. Package management that looks like an already very easy package manager (aptitude) is a bonus in my book. Installing gems will become something you most likely get very comfortable with as this is the easiest way to get the libraries you need for your Ruby installation. Read more into [RVM Gemsets](http://rvm.beginrescueend.com/gemsets/) to see where this stuff gets even cooler. _Note: If you get this error message:_

```bash
ERROR: Loading command: install (LoadError) no such file to load -- zlib
```

_take a look [at this page for installing Ruby with zlib](http://rvm.beginrescueend.com/packages/zlib/). I tripped over this nicely._
