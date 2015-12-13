---
layout: post
title: "Cytoscape Scripting with Groovy"
modified:
categories: blog
excerpt:
tags: [bioinformatics,cytoscape,groovy]
image:
  feature:
date: 2012-6-28
---

[Cytoscape](http://www.cytoscape.org/) is a network analysis package used quite often in systems biology. Cytoscape has scripting support for Python, Ruby, and Groovy.  Or at least they say there is scripting support for Groovy, but there wasn't a meaningful Groovy example that I could find.  So I made a couple of minor changes to the Ruby example, along with logging added for debugging and, behold, an example Cytoscape Groovy script:

{% highlight groovy %}
import cytoscape.Cytoscape;
import cytoscape.layout.CyLayouts;
import cytoscape.layout.LayoutAlgorithm;
import cytoscape.CytoscapeInit

errFile = new File("test.out").withWriter{w->
  w.println "CYTOSCAPE SCRIPT LOG"

  try{
    props = CytoscapeInit.getProperties()
    props.setProperty("layout.default", "force-directed")

    new File("sampleData").eachFileMatch(~/.*\.sif/){f->
      w.println "Loading $f"
      Cytoscape.createNetworkFromFile(f.canonicalPath)
      Cytoscape.getCurrentNetworkView().redrawGraph(false,true)
    }
  }catch(Exception e){
    w.println e
  }
}
{% endhighlight %}
