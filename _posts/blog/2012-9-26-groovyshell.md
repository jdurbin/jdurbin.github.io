---
layout: post
title: "RunBash: Execute Real Shell Commands from Groovy"
modified:
categories: blog
excerpt:
tags: [groovy,bash,shell]
image:
  feature:
date: 2012-9-26
---
#### RunBash
This post is about running shell commands from within Groovy, specifically bash but it is easy to adapt to other shells.  Groovy has built-in support for running commands like this:

`"ls -l".execute()`

That is about as simple as it can get and works great for many situations. However there is a key gotcha: `execute()` simply executes the given command passing it whatever else is in the string as options to the command.  The options are **not** passed through any shell (e.g. bash) for wildcard expansion or other transformations. As a result, you can *not* natively do something like:

`"ls *.groovy".execute()`

In this case, no shell sees the `*` to expand it, and so the `*` just gets passed to `ls` exactly as an argument which `ls` itself does not know how to interpret. To address this, we can create a shell process with `ProcessBuilder` and pass the command to the shell for execution. A common use case for me is to want to just pipe the shell command's output to stdout.  With some Groovy meta-object programming we can make this a method of `GString` and `String` so that you can execute any kind of string simply by calling, for example, a .bash() method on the string.  I have written a class called `RunBash` which provides this function.  With this class you can properly execute the `ls *.groovy` example above with code like `"ls *.groovy".bash()`.  You can even execute more complicated shell scripts like:

{% highlight groovy %}
"""
 for file in \$(ls); do
   echo \$file
 done
""".bash()
{% endhighlight %}

To turn on this functionality it is necessary to call `RunBash.enable()` first. So a full example using the [durbinlib](https://github.com/jdurbin/durbinlib) implementation of RunBash is:

{% highlight groovy %}
#!/usr/bin/env groovy

import durbin.util.*
RunBash.enable()

"""
for file in \$(ls); do
   echo \$file
done
""".bash()
{% endhighlight %}


#### Installation

You can obtain `RunBash` with [durbinlib](https://github.com/jdurbin/durbinlib), which is my kitchen-sink of everday classes and scripts.  Simply clone, ant build, and add to `CLASSPATH` and `PATH` like:

{% highlight bash %}
git clone git://github.com/jdurbin/durbinlib.git
cd durbinlib
ant install
export PATH=$PATH:pathtodurbinlib/scripts/ 
export CLASSPATH=pathtodurbinlib/target/jar/*
{% endhighlight %}

####Source

To get an idea how this is implemented in Groovy code, a stripped-down but functional version of the class is shown below:

{% highlight groovy %}
import java.io.InputStream;

class RunBash{
  
  static boolean bEchoCommand = false;
  
  // Add a bash() method to GString and String with meta-object programming
  // This is why it's necessary to call an enable function in your script. 
  static def enable(){
    GString.metaClass.bash = {->
      RunBash.bash(delegate)
    }    
    String.metaClass.bash = {->
        RunBash.bash(delegate)
    }    
  }
  
  static def bash(cmd){
    cmd = cmd as String

    // create a process for the shell
    ProcessBuilder pb = new ProcessBuilder("bash", "-c", cmd);
    pb.redirectErrorStream(true); // use this to capture messages sent to stderr
    Process shell = pb.start();
    shell.getOutputStream().close();
    InputStream shellIn = shell.getInputStream(); // this captures the output from the command

    // at this point you can process the output issued by the command
    // for instance, this reads the output and writes it to System.out:
    int c;
    while ((c = shellIn.read()) != -1){
      System.out.write(c);
    }

    // wait for the shell to finish and get the return code
    int shellExitStatus = shell.waitFor(); 

    // close the stream
    try {
      shellIn.close();
      pb = null;
      shell = null;
    } catch (IOException ignoreMe) {}
  }
}
{% endhighlight %}