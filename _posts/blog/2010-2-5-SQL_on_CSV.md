---
layout: post
title: "Executing Arbitrary SQL on CSV Files"
modified:
categories: blog
excerpt:
tags: [SQL,CSV,groovy,h2,database]
image:
  feature:
date: 2010-2-5
---

My life is full of comma separated value files. A common occurrence is to be given a csv file of data and want to explore it a little bit to see what is there, or to extract several columns of data out of the csv file and create a new csv file, or grab only certain columns of data and certain rows of data and create a new csv file. Quite often I find that I even need to join one csv file with another csv file. Each time I'm faced with this task I find that what I want is to do execute sql statements on the csv file itself. After ages of writing one-off scripts to process files like these, I finally looked into actually treating csv files as databases. It turns out that the [h2 database](http://www.h2database.com/html/main.html) engine has good support for both in-memory databases and csv file importing. With Groovy, it's very nice. Simply [download the h2 database jar](http://www.h2database.com/html/download.html) and drop it in your ``~/.groovy/lib/ directory``. Then you can write code like this:


{% highlight groovy %}
#!/usr/bin/env groovy

import groovy.sql.Sql
import org.h2.Driver

// Create an h2 jdbc in-memory database,
// calling it what you like, here db1
def db = Sql.newInstance("jdbc:h2:mem:db1","org.h2.Driver")

// Create a table from your csv file... 
db.execute("create table people as select * from csvread('people.csv')")

// Execute a normal sql query on this table...
db.eachRow("select * from people where age < 40"){row->
 println row
}
{% endhighlight %}


If you want to create a new csv file from your query, you will have to do a tiny bit more work. First you will need to extract the column headings of your query from the metadata, and then you will need to collect the rows into comma separated list. It's easy boiler-plate once you know how. Here is an example:

{% highlight groovy %}
#!/usr/bin/env groovy 

import groovy.sql.Sql
import org.h2.Driver

// Create an h2 jdbc in-memory database, 
// calling it what you like, here db1 
def db = Sql.newInstance("jdbc:h2:mem:db1","org.h2.Driver")

// Create a table from your csv file...  
db.execute("create table people as select * from csvread('people.csv')")

// Get the column headings...
db.eachRow("select * from people where age < 40 limit 1"){row->
  meta = row.getMetaData()
  numCols = meta.getColumnCount()
  headings = (1..numCols).collect{meta.getColumnLabel(it)}
  println headings.join(",")
}

// Execute a normal sql query on this table...
db.eachRow("select * from people where age < 40"){row->
  meta = row.getMetaData()
  numCols = meta.getColumnCount()
  vals = (0..<numCols).collect{row[it]}
  println vals.join(",")
}
{% endhighlight %}

The h2 database engine pretty quick too. I can query a 100k file (both read it in as a table and do select on it) in about 5 seconds using code like that above.

Finally, to do a join on two csv files, you only need to create the two files and execute the sql, like

{% highlight groovy %}
#!/usr/bin/env groovy 

import groovy.sql.Sql
import org.h2.Driver

// mem says in-memory db, and call it what you like, here db1 
def sql = Sql.newInstance("jdbc:h2:mem:db1","org.h2.Driver")

// Create the first table...  
sql.execute("create table people as select * from csvread('people.csv')")

// Create the second table...
sql.execute("create table children as select * from csvread('children.csv')")

sql.eachRow("""
select people.name,children.child,people.age
from people,children 
where people.name=children.name and people.age > 40
"""){result->
  
  meta = result.getMetaData()
  cols = meta.getColumnCount()
  vals = (0..<cols).collect{result[it]}
  println vals.join(",")  
}
{% endhighlight %}
