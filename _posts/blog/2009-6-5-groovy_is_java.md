---
layout: post
title: "Groovy is Java"
modified:
categories: blog
excerpt:
tags: [bioinformatics,groovy,java]
image:
  feature:
date: 2009-05-06
---

[Groovy](http://groovy-lang.org) is groovy. At least for me. Groovy is also Java.  A kind of quick and dirty Java all dripping with syntatic sugar. If you know Java you essentially know 80% of what there is to know about Groovy because underneath it's all Java objects and the syntax is roughly a superset of Java. Most Java programs will run, unaltered, as Groovy programs. Although Groovy's dynamic features, powerful language constructs, and other goodness will grow on you, a Java programmer can approach Groovy initially as if it were simply a kind of Java that can be executed without compiling and with the option to soft focus some of the details. 

For example, here is a dumb little Groovy script that I wrote to convert a comma separated file into a tab separated file:

{% highlight groovy %}
#!/usr/bin/env groovy

for(line in System.in.readLines()){
  bits = line.split(",");
  for(bit in bits){
    print bit+"\t";
  }
  println "";
}
{% endhighlight %} 

This is something I wrote on the first day I was learning Groovy so it is not idiomatic Groovy. Groovy can be much more succinct than this. A Perl or Python person might prefer a command line one-liner like:

`cat test.tab | groovy -pe '(line =~/\t/).replaceAll(",")'`

Nonetheless, this little script shows how one can come to Groovy gently from Java. First, the program above is a complete Groovy program. If it were Java, there'd have to be more, the declaration of a containing class, for example. Groovy provides an implicit containing class with the name of the file. You'd also have to be more fastidious about declaring the types of variables. Groovy works out types during runtime. The types Groovy eventually assigns to variables will be Java types: line and bits, for example, will resolve to `java.lang.String`, and all the functionality of java.lang.String is there for you just like you remember it. Also, unlike a Java program, Groovy scripts don't need compiling (if you need to, though, you can compile Groovy to a .class file to use in a Java program). You can just run the file like a Python script, for example:

`cat data.txt | comma2tab  > data.tab`

So you can see that you get that sort of Python-like hack-together-something-quick quality: no compiling, direct execution of the source file (via shebang or the command line interface), rapid edit/test cycle, and stripped down verbosity. Suppose I also wanted to add a date tag to each line. When I was doing my scripting with Python I'd have to grind to a halt while I looked up various Python date APIs. Obviously, if 99% of my code were scripting Python I'd probably already know that, but my code is more like 10% C++, 65% Java, and 25% scripting. As a result, I'm much more likely to already know the Java API for something than the Python API. With Groovy I can add date functionality without having to learn any new API's, I can just use the Java one I already know: 

{% highlight groovy %}
#!/usr/bin/env groovy  
  
date = new Date();  
for(line in System.in.readLines()){  
  println date.toString()+"\t";  
  bits = line.split(",");  
  for(bit in bits){  
    print bit+"\t";  
  }  
  println "";  
}
{% endhighlight %}

That print is, by the way, `System.out.println`, it' just lets you use the shorthand if you like. You don't have to though. With a simple import, you can use any of your own Java classes just as easily. Drop any external library into your CLASSPATH, or a jar into your ``~/groovy/lib`` directory, and you have ready access to that functionality too. You can even hack together simple GUI's in just a few lines of code.

Given that I have worked on several pure-Java projects in the past few years, Groovy really fills a niche for me. Working in bioinformatics I have a lot of need for scripting, to create quick and dirty one-off programs to crunch some data or do some bulk operation, something someone wants today and will never use again. I also have a fair bit of need for production quality webpages, for computation intensive algorithms, and for GUI visualization tools. The combination of Groovy+Java nicely fills all of these needs for me. I can write my high quality webpages in Java (or [Grails](http://www.grails.org/)), my computation intensive algorithms in Java (which has become performance competitive in the past few years, typically within 2x of C[^1] or better), my GUI visualization tools in Groovy/Java/Swing, and my one-off scripts and pipeline glue in Groovy. One set of libraries and API's to master to do all my work. This gives me the ability to share code across all the kinds of work I do. All my code is cross platform. It's like having my cake and eating it too! 

If you know Java, you owe it to yourself to try Groovy. If you find yourself writing one kind of software in one language, another in a different language, you owe it to yourself to try Groovy. The rest of you, you should probably try it too. 

[^1]: <http://benchmarksgame.alioth.debian.org/u64q/java.html>