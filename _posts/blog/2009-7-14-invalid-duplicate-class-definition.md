---
layout: post
title: "Create/open OmniOutliner notebooks from command line"
modified:
categories: blog
excerpt:
tags: [groovy]
image:
  feature:
date: 2009-07-09
---

>Invalid duplicate class definition...One of the classes is a explicit generated class using the class statement, the other is a class generated from the script body based on the file name. Solutions are to change the file name or to change the class name. 

When I was first learning Groovy I would get this error from time to time. It was puzzling to me because sometimes I'd get this error, and sometimes it would seem like the same situation and I wouldn't. It seemed quite random to me when the error would crop up. When it did, I'd usually just rename the class and go on, resolving to figure it out later. To save you the trouble, here's what is happening. 

Groovy has two ways to treat a .groovy file: either as a *script*, or as a *class definition* file. If it is a script you can not have a class by the same name as the file. If it is a class definition file you can. It is very easy to tell whether a .groovy file is going to be treated as a script or as a class definition file. If there is any code outside a class statement in the file (other than imports), it is a script. What is happening is that if there is any code to be executed in the file then Groovy needs a containing class for that code. Groovy will implicitly create a containing class with the name of the file. So if you have a file called `Grapher.groovy` that has some code in it that isn't inside a class definition, Groovy will create an implicit containing class called `Grapher`. This means that the script file `Grapher.groovy` can not itself contain a class called `Grapher` because that would be a duplicate class definition, thus the error. If, on the other hand, all you do in the file `Grapher.groovy` is define the class `Grapher` (and any number of other classes), then Groovy will treat that file as simply a collection of class definitions, there will be no implicit containing class, and there is no problem having a class called `Grapher` inside the class definition file `Grapher.groovy`. 

It's worth mentioning that the script version of `Grapher.groovy` will be compiled into a class called `Grapher` that extends `groovy.lang.Script`. In the other case, when `Grapher.groovy` merely defines classes, one of which is `Grapher`, that Grapher class will be compiled into a class that implements `groovy.lang.GroovyObject`. 

I'm sure this is all explained somewhere in the Groovy documentation, but it didn't soak in to me until I read this Nabble post from which I extracted this explanation.

UPDATE: The text of this error message has changed (at least in some cases) to be a bit more informative. Now it reads:

>One of the classes is an explicit generated class using the class statement, the other is a class generated from the script body based on the file name. Solutions are to change the file name or to change the class name.
The underlying mechanics behind this error are the same.