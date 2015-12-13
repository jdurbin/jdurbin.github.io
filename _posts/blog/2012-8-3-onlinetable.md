---
layout: post
title: "OnlineTable: Class to access csv files one row at a time by named columns."
modified:
categories: blog
excerpt:
tags: [groovy,csv]
image:
  feature:
date: 2012-8-3
---

####OnlineTable

Here is a handy class I use to simplify accessing a CSV or TAB file one line at a time. I feel like I've seen this somewhere else also. I hope it's not just in [GINA](https://www.manning.com/books/groovy-in-action-second-edition)!  Anyway, sometimes the simplest things are the most handy so it bears repeating even if it is. Suppose you have a file, `beer.csv`, that has a header describing the columns followed by rows of data like:

    brand,price,calories,alcohol,type,domestic
    LeinenkugelsRed,4.79,160,5.0,1,1
    SamuelAdamsBoston,5.96,160,4.9,1,1
    GeorgeKilliansIrishRed,4.70,162,4.9,1,1
    RedWolf,4.11,157,5.5,1,1
    Becks,5.83,148,4.3,3,0
    PilsnerUrquell,7.80,160,4.1,3,0

OnlineTable allows you to go through the file a row at a time and parse it like this:

{% highlight groovy %}
new OnlineTable("beer.csv").eachRow{row->
  println "brand: "+row.brand
  println "calories:"+row.calories
}
{% endhighlight %}

Or, if you prefer the map notation, you can use it like:
{% highlight groovy %}
new OnlineTable("beer.csv").eachRow{row->
  println "brand: "+row['brand']
}
{% endhighlight %}

OnlineTable automatically inspects the header to determine if it is a CSV or TAB delimited file. 

#### Installation

Part of my kitchen-sink of everday functionality, [durbinlib](https://github.com/jdurbin/durbinlib).  Simply clone and ant build, and add to CLASSPATH and PATH:

{% highlight bash %}
git clone git://github.com/jdurbin/durbinlib.git
cd durbinlib
ant install
export PATH=$PATH:pathtodurbinlib/scripts/ 
export CLASSPATH=pathtodurbinlib/target/jar/*
{% endhighlight %}

#### Source

An early version of the source, illustrating how it is implemented:

{% highlight groovy %}
/***********************************
* Support for accessing a table one row at a time.
*
*/
class OnlineTable{
	String fileName
	
	def OnlineTable(String f){
        fileName = f
      }     
      
	def eachRow(Closure c){
	  new File(fileName).withReader{r->
	    def headingStr = r.readLine()
	    def sep = determineSeparator(headingStr)
	    def headings = headingStr.split(sep)
	    r.eachLine{rowStr->
	      def rfields = rowStr.split(sep)
	      def row = [:]
	      rfields.eachWithIndex{f,i->row[headings[i]]=f}
	      c(row)
	    }
	  }
	} 
	      
	def determineSeparator(line){
	  def sep;
	  if (line.contains(",")) sep = ","
	  else if (line.contains("\t")) sep = "\t"
	  else {
	    System.err.println "File does not appear to be a csv or tab file.";
	    throw new RuntimeException();
	  }
	  return(sep);
	}   
}
{% endhighlight %}