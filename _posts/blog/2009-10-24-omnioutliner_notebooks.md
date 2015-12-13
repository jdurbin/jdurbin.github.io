---
layout: post
title: "Create/open OmniOutliner notebooks from command line"
modified:
categories: blog
excerpt:
tags: [OmniOutliner,groovy]
image:
  feature:
date: 2009-10-24
---

I like to use [OmniOutliner](http://www.omnigroup.com/applications/omnioutliner/) as a lab notebook, and I like to be able to create/open notebooks from the command line. The OS X 'open' command will open an existing file, but I didn't see any obvious way to run OmniOutliner from the command line and tell it to create a new file. So I created an empty OmniOutliner file and dropped in ~/Documents/template, then wrote a little script to copy that template and open it if it doesn't exist, or open the pre-existing file if it does. Simple and effective. 


{% highlight groovy %}
#!/usr/bin/env groovy 
/********************************************
*  A simple little script to open up a file in OmniOutliner
*  creating a default file if the file does not already exist. 
* 
*  Run it like:
* 
*  omni newNotebook
*/

def omniFile
if (args[0].contains(".oo3")){
  omniFile = args[0]
}else{
  omniFile = "${args[0]}.oo3"
}

// Get our home directory..
env = System.getenv()
home = env['HOME']

// If there is no omni outline file there, copy a template over...
if (!(new File(omniFile).exists())){
  "cp -r $home/Documents/templates/OOTemplate.oo3/ ${omniFile}".execute()
}

// Open the desired file..
"open ${omniFile}".execute()
{% endhighlight %}
