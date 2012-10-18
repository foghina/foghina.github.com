---
layout: post
title: "How to fuck up with find"
date: 2012-10-04 20:32
comments: true
categories: [Shell, "*nix"]
---

Welcome to the tutorial on how to successfully fuck up with `find`, the command line tool. In case you're not familiar with the `find` command, [RTFM](http://linux.die.net/man/1/find), it's freaking beautiful. However, in the hands of a non-RTFM-ing user (like me) it can be quite destructive. Let me demonstrate.

I wanted to use `find` to delete a bunch of files from a hierarchy, based on a simple name rule. So I thought I should use [`xargs`](http://linux.die.net/man/1/xargs) in combination with `find` to delete them. After a quick lookup of the `-print0` and `-name` arguments of the `find` command, I concluded that it would be a good idea to run the following command:

``` console
$ find . -print0 -name example | xargs -0 rm
```

Can you guess what it did? Let me help you: it deleted all the files under `.`. The directories are still there, but they're not any help, are they? Now, fortunately for me, this was in a git repository that I had just pushed to a remote, so I was able to just clone it again.

In order to understand what happened, I had to actually read the man page carefully and pay attention. Apparently, everything that comes after the path (in my case `.`) on the command line is treated as an expression by `find`, where every argument evaluates to true or false, and is ORed with the next one. The `-print0` argument always returns true, because it's only meant to change the output, not the actual filtering. Because of this, my `-name` argument was completely ignored.

To conclude, the correct command would have been:

``` console
$ find . -name example -print0 | xargs -0 rm
```

This way, the `-name` argument will take precedence over `print0`. However, because I actually spent time reading the manual this time, I discovered `find` also has a `-delete` argument, so `xargs` is not actually needed at all:

``` console
$ find . -name example -delete
```

As I was saying, freaking beautiful.