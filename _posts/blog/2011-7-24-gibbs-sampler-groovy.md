---
layout: post
title: "Gibbs sampler in Groovy"
modified:
categories: blog
excerpt:
tags: [gibbs sampling,mcmc,groovy]
image:
  feature:
date: 2011-7-24
---

I recently read a couple of nice articles by Darren Wilkinson about implementing MCMC in various languages. The posts are [here](http://darrenjw.wordpress.com/2010/04/28/mcmc-programming-in-r-python-java-and-c/) and [here](http://darrenjw.wordpress.com/2011/07/16/gibbs-sampler-in-various-languages-revisited/).  Wilkinson apparently uses, or is considering using, Python for a lot of prototyping and C for a lot of his actual MCMC runs. However, since he feels that Java is in some ways nicer than C, and almost as fast, he has been using Java for some of the final MCMC runs.  I thought I'd see how Groovy performed on this task. 

A Groovy version of Wilkinson's Gibbs sampler is given here: 

{% highlight groovy %}
#!/usr/bin/env groovy 
import static java.lang.Math.*
import cern.jet.random.tdouble.engine.*
import cern.jet.random.tdouble.*

N=20000
thin=500
rngEngine= new DoubleMersenneTwister(new Date())
rngN= new Normal(0.0,1.0,rngEngine)
rngG= new Gamma(1.0,1.0,rngEngine)
x=y=0.0
println("Iter x y")
for(i in 1..N){
  for(j in 1..thin){
    x=rngG.nextDouble(3.0,y*y+4)
    y=rngN.nextDouble(1.0/(x+1),1.0/sqrt(x+1))
  }
  println("$i $x $y")
}
{% endhighlight %}

You run this like: `gibbs.gv > data.tab`

On a Macbook Pro 2.2 GHz Intel Core 2 Duo (~2007), comparing Wilkinson's Java (1.6.1) and Python (2.7.2) versions with my Groovy (1.8.0) version, I get the following times: 

* java: 5.23 sec
* groovy: 16.5s
* python: 1m49s

Not bad for a dynamic scripting language! Now, there are lots of things one can do to speed up the Python code, from using pypy to calling native functions and so on, so this is nowhere close to optimal for Python but... if you're going to ultimately translate your prototype into Java anyway why bother with Python at all? It's easy to rapid-prototype in Groovy and then, if you ever need the extra performance you can perform the simple translation from Groovy to Java.  The libraries and almost all details will be the same which greatly reduces learning and debugging time.  Moreover, at one third the speed of Java, in many situations you won't ever have the need to bother with the speed-up and can just stop at the Groovy rapid-prototype. 

---

*Update 12/2015:*  I have re-run the Gibbs sampler using a newer 2015 MacBook Pro and newer versions of all libraries, including using ParallelColt.  Now, using Groovy Version: 2.4.5, Java Version 1.8.0_31, and Python 2.7.10 :: Anaconda 2.2.0,  I get the following times:

* java:	  2.3s
* groovy: 5.8s
* python: 30.8s

Again, this is out-of-the-box time.  I'm sure we could speed it up with pypy or other tricks.  Still, a 5x speed difference is pretty significant and illustrates why prototyping in Groovy can often leave you with a solution that is good enough, with an easy 2x speed improvement if you need it by performing the simple Java translation. 