---
comments: true
date: 2012-04-22 12:00:00
layout: post
slug: beware-of-the-javascript-reference
title: Beware of the JavaScript reference!
wordpress_id: 20
categories:
- Coding
tags:
- javascript
---

As you (hopefully) know, variables in JavaScript are actually references. As you (most surely) know, references can bite you in the ass in the most unexpected places, so here's one of them. Say you have an object that you continuously change (e.g. recursively) and want to see how it "evolves" by calling console.log() on it. You would expect to see "snapshots" of the object logged, right? Wrong. For example, try running this code in a console (**Note:** I have only tried this in Chrome, not sure how other browsers behave):

``` javascript
var mutant = {"name": "Leela Turanga"};
console.log(mutant);
mutant["characteristic"] = "One eye";
```

Now, expand the Object and voilà, you'll see both properties. This can be very confusing with more complicated objects. If you're too lazy to run that code, here's proof:

{% img /images/posts/javascript-reference.png Screenshot of Chrome console %}

As a solution, you can clone the object before logging it. [Here](http://stackoverflow.com/questions/122102/what-is-the-most-efficient-way-to-clone-a-javascript-object) are a few suggestions for how to do that.
