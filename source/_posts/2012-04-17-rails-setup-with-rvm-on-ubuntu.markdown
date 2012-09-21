---
comments: true
date: 2012-04-17 00:39:45
layout: post
slug: rails-setup-with-rvm-on-ubuntu
title: Rails setup with RVM on Ubuntu
wordpress_id: 7
categories:
- Coding
tags:
- guide
- rails
- ruby
---

Given how developer friendly as Ruby on Rails is, getting a development environment up and running can be surprisingly tricky. I will describe the process that I consider to be "best practice" here, step by step. Although my guide will be focused on Ubuntu, the general idea should be the same on other distros as well. Also, please take note of the date of this post. If you're reading this even a few months after I wrote this, don't be surprised if your mileage will vary.

<del>First off, we're going to need Ruby, right? So let's install that, and also throw in curl (we're going to need it in a bit):</del>

**Update:** actually, you don't need Ruby installed on your system at all. Installing curl is enough here:

``` console
$ sudo apt-get install curl
```

Next, we're going to install [RVM](https://rvm.io/). RVM is, at the website puts it, a Ruby Version Manager. Basically, it allows you to install multiple versions of Ruby in your home folder and use them seamlessly (as if they were the default Ruby in the system). This is useful for us because the version of Ruby Ubuntu installed is 1.8.7 (as of this writing), while the latest stable version of Ruby is 1.9.3. RVM also allows us to keep separate collections of gems (called gemsets, for obvious reasons). This is very useful when working with multiple projects that may have conflicting dependencies.

Installing RVM is dead simple. Just run the command under **Quick Install**:

``` console
$ curl -L get.rvm.io | bash -s stable
```

**Warning:** the above command might not be the latest one. Make sure to double check with the RVM website.

Now, close the current terminal and open a new one. This is so that RVM can load into the new bash session. Now, let's see what RVM suggests we should install:

``` console
$ rvm requirements

Requirements for Linux ( DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=11.10
DISTRIB_CODENAME=oneiric
DISTRIB_DESCRIPTION="Ubuntu 11.10" )

NOTE: 'ruby' represents Matz's Ruby Interpreter (MRI) (1.8.X, 1.9.X)
             This is the *original* / standard Ruby Language Interpreter
      'ree'  represents Ruby Enterprise Edition
      'rbx'  represents Rubinius

bash >= 4.1 required
curl is required
git is required (>= 1.7 for ruby-head)
patch is required (for 1.8 rubies and some ruby-head's).

To install rbx and/or Ruby 1.9 head (MRI) (eg. 1.9.2-head),
then you must install and use rvm 1.8.7 first.

Additional Dependencies:
# For Ruby / Ruby HEAD (MRI, Rubinius, & REE), install the following:
  ruby: /usr/bin/apt-get install build-essential openssl libreadline6 libreadline6-dev curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev automake libtool bison subversion

# For JRuby, install the following:
  jruby: /usr/bin/apt-get install curl g++ openjdk-6-jre-headless
  jruby-head: /usr/bin/apt-get install ant openjdk-6-jdk

# For IronRuby, install the following:
  ironruby: /usr/bin/apt-get install curl mono-2.0-devel
```

Install the dependencies suggested by RVM for Ruby (see the marked lines above and make sure to use sudo).

Good. Now it's time to install Ruby! **Again.** The next command will install the latest stable Ruby into your home directory (so the system is not affected). Beware, it will take a long time to complete, as it downloads the Ruby source code and compiles it!

``` console
$ rvm install ruby
```

Now, if the command above finishes with a message similar to:

    RVM is not a function, selecting rubies with 'rvm use ...' will not work.
    Please visit https://rvm.io/integration/gnome-terminal/ for a solution.

Then do visit that URL and follow the instructions there. Finished? Good. Open a new terminal, **again**. Now type:

``` console
$ rvm use ruby
```

If all went well, you should see a green message telling you which Ruby is being used. Next, you should create a gemset (we talked about them in the beginning) for your new project, and also switch to it:

``` console
$ rvm gemset create myproject && rvm gemset use myproject
```

You finally have your environment set up. Next step is to install the rails gem:

``` console
$ gem install rails --no-rdoc --no-ri
```

**Note:** the --no-rdoc --no-ri parameters are passed so that it doesn't waste time installing docs that are available on the Internet anyway. If you wish to have them installed for some reason, just omit those parameters.

Good, now you're ready to create your new Rails app! Simply type:

``` console
$ rails new myproject
```

To run the new app:

``` console
$ cd myproject/
$ rails server
```

You just got an error now, didn't you? Did it sound like this?

``` console
in `autodetect': Could not find a JavaScript runtime. See https://github.com/sstephenson/execjs for a list of available runtimes. (ExecJS::RuntimeUnavailable)
```

What's happening is that Rails needs a JavaScript runtime (well, duh, it says so right in the error message!). What it needs it for, from what I can tell, is the new CoffeeScript (and LESS?) functionality added in the asset pipeline in Rails 3.1. To fix this, just require a JavaScript runtime gem in your app. We're going to use one called "therubyracer". Open up the file called "Gemfile" and add this line to it:

``` console
gem 'therubyracer'
```

Now, in order to actually install that gem, just run:

``` console
$ bundle
```

in your project's directory. The bundle command basically looks in your Gemfile and installs / updates dependencies based on it. Running the app should now actually finally ultimately work:

``` console
$ rails server
=> Booting WEBrick
=> Rails 3.2.3 application starting in development on http://0.0.0.0:3000
=> Call with -d to detach
=> Ctrl-C to shutdown server
[2012-04-17 00:24:51] INFO  WEBrick 1.3.1
[2012-04-17 00:24:51] INFO  ruby 1.9.3 (2012-02-16) [x86_64-linux]
[2012-04-17 00:24:51] INFO  WEBrick::HTTPServer#start: pid=4595 port=3000
```

Now, every time you open a new terminal and intend to run the project, you will have to:

``` console
$ cd myproject/
$ rvm use ruby
$ rvm gemset use myproject
$ rails server
```

If you want to skip the two `rvm` commands, you can use an `.rvmrc` file. That, however, is beyond the scope of this post, and is documented [here](https://rvm.io/workflow/rvmrc/).

As I was saying, surprisingly tricky. Sure, you could have used the Ruby and Rails provided by your package manager, but given the rapid development of both, you would be several major versions behind and lacking many features. Also, managing multiple projects is now a breeze. Just install the required Ruby (some projects might rely on older versions of it), create a new gemset for it, install dependencies, run.

Whew!
