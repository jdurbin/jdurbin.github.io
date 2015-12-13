---
layout: post
title: "viewtab: A Fast Big Data Spreadsheet"
modified:
categories: blog
excerpt:
tags: [big data,csv,groovy,plotting]
image:
  feature:
date: 2012-6-5
---

### The need for a csv/tab viewer

Do you have TSV/CSV files that you'd like to load into a spreadsheet and explore but they are just too large for Excel or Numbers?  `viewtab`, which can be obtained by installing [durbinlib](https://github.com/jdurbin/durbinlib) is the tool for you! 

In a [previous post]({{ site.url }}/blog/csvsql/) I showed 'csvsql' which is one tool in my arsenel of TSV/CSV file handling.  Today's posts shows another tool.  The problems this tool addresses is the need to figure out what is in some file, what kind of text or numbers, to explore the ranges of the numbers with histograms or scatter plots, etc.   I often get data files from scientists or from the supplements of journal papers without any clue what is in the file.   Often I'm even faced with trying to figure out which of many files might contain information that I need. One can `cat` the file, of course, but if the file has many columns a simple cat quickly becomes illegible.  There are fancier things one can do with `cat` such as:

`cat test.csv  | column -s, -t | less -#2 -N -S`

This gives you formatted columns that you can scroll through, but I find it clunky.  The columns can not be easily sorted, moved around, and so on.  Importing the file into a spreadsheet is another option, but for some reason every spreadsheet out there takes an eon to start up. Some of them won't take csv or tab files from the command line so that you have to launch the spreadsheet then navigate to the file in question which is actually one of the biggest drawbacks of spreadsheets for my use case. Even worse, most spreadsheets simply choke on what are for me modest sized data files (say, a few tens of thousands of rows with a few hundreds of columns).  

### viewtab
After facing this irritation dozens of times, I decided to see if I could hack together a csv/tab viewer that would offer the advantages of a spreadsheet with the speed and command-line convenience of cat. The resulting script is run like this:

`viewtab testfile.csv`
 
 It takes about 6 seconds to load a 9803 x 296 csv file and display it. Compare that with Apple's Numbers which takes 3 seconds to launch, 3-4 seconds for me to find the file, tries to load it for about 8 seconds and finally tells you the file is too big and simply gives up! 
 
 
 The results of viewtab look like this (here the expression data from a 2012 breast cancer paper by van de Vijver):
 
 
 ![viewtabimage]({{ site.url }}/images/viewtab.png)

`viewtab` also allows you to select a row or column (with right-click menu) and get a histogram of that row or column, as seen here:

![viewtabhistimage]({{ site.url }}/images/viewtabhist.png) 

You can even select two rows/columns and generate a scatter plot of one vs the other.  Additional analysis functions are in the works. 

### Installation

`viewtab`, `csvsql`, and a number of other very useful scripts are bundled with [durbinlib](https://github.com/jdurbin/durbinlib), which is my kitchen sink of libraries and scripts I use every day in my bioinformatics work.  The easiest way to install the library is to clone durbinlib, build it with [ant](http://ant.apache.org), and then add the `scripts` directory to your path. 

{% highlight bash %}
git clone git://github.com/jdurbin/durbinlib.git
cd durbinlib
ant install
export PATH=$PATH:pathtodurbinlib/scripts/ 
{% endhighlight %}

I highly recommend that you set an environment variable to give Groovy/Java the option to use a lot of RAM:

`export JAVA_OPTS='-Xmx3000m'`

Durbinlib requires fairly recent versions of [Java](https://www.java.com/en/download/),[Groovy](http://groovy-lang.org), and [Ant](http://ant.apache.org).  All other dependencies are bundled into the Git repository. 

### Complete first version of the code

To illustrate how easy it is to hack together a GUI app with Groovy, below is the complete source for the first version of `viewtab` (before adding plotting and other extras). 

{% highlight groovy %}
#!/usr/bin/env groovy 

import groovy.swing.SwingBuilder
import javax.swing.JTable
import javax.swing.*
import javax.swing.table.AbstractTableModel;
import durbin.util.*

err = System.err

class TableModel extends AbstractTableModel{  
  Table dt;
  
  TableModel(Table table){dt = table;}
  
  public String getColumnName(int col) {return(dt.colNames[col]);}
  public String getRowName(int row){return(dt.rowNames[row]);}  
  public int getRowCount() { return(dt.rows())}
  public int getColumnCount() { return(dt.cols()) }
  public String getValueAt(int row, int col) {return(dt.get(row,col))}  
  public Class getColumnClass(int c) {return(dt.get(0,c).getClass())}   
  public boolean isCellEditable(int row, int col){return false;}  
  public void setValueAt(Object value, int row, int col) {}
}

fileName = args[0]

// Crudely determine if it's a tab or csv file
new File(fileName).withReader{r->
  line = r.readLine()
  if (line.contains(",")) sep = ","
  else if (line.contains("\t")) sep = "\t"
  else {err.println "File does not appear to be a csv or tab file.";System.exit(1)}
}

dt = new Table(fileName,sep,bFirstRowInTable=true)

err.print "Creating gui table..."
dtm = new TableModel(dt)

swing = new SwingBuilder()
frame = swing.frame(title:fileName,defaultCloseOperation:JFrame.EXIT_ON_CLOSE){
  scrollpane = scrollPane {           
    thetab = table(autoResizeMode:JTable.AUTO_RESIZE_OFF, autoCreateRowSorter:true){
      tableModel(dtm) 
    }             
  }
}
err.println "done."
frame.pack()
frame.show()
{% endhighlight %}

