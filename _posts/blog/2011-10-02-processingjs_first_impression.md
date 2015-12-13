---
layout: post
title: "Processing.js First Impression"
modified:
categories: blog
excerpt:
tags: [canvas,html5,javascript,processing,processing.js]
image:
  feature:
date: 2011-10-02
---


I've been playing around with [Processing.js](http://processingjs.org/download/) to produce visualizations of some genomics data. The Java based Processing language/environment was originally developed by the noted data visualization guru [Ben Fry](http://benfry.com) and graphic artist [Casey Reas](reas.com). Processing.js is a port of the Processing language and libraries to javascript and the HTML 5 canvas by none other than [John Resig](http://ejohn.org), the creator of the [jQuery](http://jquery.com) javascript library. I was skeptical at first that Processing.js would be anything but a pale shadow of it's Processing predecessor, but after using it for a bit I have to say that I am impressed.  Most of the Processing sketches one can find scattered around the web and on such sites as [Open Processing](http://www.openprocessing.org/) just work. Here, for example, is a sketch from the examples on Processing.org  called [reflection2](http://processing.org/learning/topics/reflection2.html). 

<i>Click on image to launch another ball. Once the canvas element has focus, type 'r' to draw a new ground</i>

{% assign lvl = page.url | append:'X' | split:'/' | size %}
{% capture relative %}{% for i in (3..lvl) %}../{% endfor %}{% endcapture %}

{% include processingtest1.html %}

The only requirement for this sketch to work is a browser that supports the HTML5 canvas. While it's not close to the speed of Processing compiled to JVM bytecode, the performance isn't bad at all and seems more than adequate for a wide range of visualization tasks. This is a testimony to the performance work that has been going into browser javascript engines as well as to the efforts put into optimizing processing.js itself. 

