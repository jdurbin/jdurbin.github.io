---
layout: post
title: "Weka: getting predictions from cross validation"
modified:
categories: blog
excerpt:
tags: [groovy,weka,wekaMine]
image:
  feature:
date: 2009-10-10
---

A [common question](http://forums.pentaho.org/showthread.php?t=62182) in [Weka](http://www.cs.waikato.ac.nz/ml/weka/) forums  is how to keep track of instances with names. Weka does not have a name field for instances, so to keep track of instances one has to create a string ID attribute that has the name of each instance. The catch, though, is that most classifiers don't work with string attributes, and you wouldn't want to classify on the ID anyway. The official solution then is to delete the ID attribute before calling the classifier. Of course, if you delete the ID, you loose the names for your instances! Oof! One solution is to use the meta.FilteredClassifier classifier with the RemoveType filter as the filter. When you hand a FilteredClassifier off to Evaluation, it will apply the filter before sending it to the classifier, but will keep track of the relationship between the source Instances (with the ID) and the filtered set sent to the classifier. Great. Now what if you want to know how instances were classified during your cross-validation? The API for extracting those classifications is not obvious, but it's easy enough once you know where to look. In Evaluate.crossValidateModel() you pass in a StringBuffer to hold the predictions. This can then be parsed to obtain the predictions and the instance names they go with. Source code to do this below: 

{% highlight groovy %}
#!/usr/bin/env groovy 

import weka.core.*
import weka.core.converters.ConverterUtils.DataSource
import weka.filters.unsupervised.attribute.RemoveType
import weka.classifiers.*
import weka.classifiers.meta.FilteredClassifier
import weka.classifiers.evaluation.*;

import java.util.Random

arffName = args[0]

// Read arff file...
data = DataSource.read(arffName)

// Pick out the class attribute..
data.setClassIndex(data.numAttributes() -1)
  
// Create a classifier from the name...
// By using filtered classifer to remove ID, the cross-validation
// wrapper will keep the original dataset and keep track of the mapping 
// between the original and the folds (minus ID). 
classifier = 
"""weka.classifiers.meta.FilteredClassifier 
      -F weka.filters.unsupervised.attribute.RemoveType 
      -W weka.classifiers.misc.HyperPipes"""

options = Utils.splitOptions(classifier)
classname = options[0]
options[0] = ""
classifier = Classifier.forName(classname,options) 

// Perform cross-validation of the model..
eval = new Evaluation(data)
predictions = new StringBuffer()
eval.crossValidateModel(classifier,data,cvFolds = 5,
  new Random(1),predictions,
  new Range("first,last"),false)

lines = predictions.toString().split("\n")
  
// Output of predictions looks like:  
// inst#     actual  predicted error prediction (ID)
//     1      1:low      1:low       1 (P1)
//     2     2:high      1:low   +   0.5 (P6)
//     3     2:high     2:high       1 (P0)
lines[1..-1].each{line->
  // Parse out fields we're interested in..      
  m = line =~ /\d:(\w+).*\d:(\w+).*\((\w+)\)/
  actual = m[0][1]
  predicted = m[0][2]
  sample = m[0][3]
  println actual+"\t"+predicted+"\t"+sample+"\t"+!line.contains("+")
}
{% endhighlight %}

[wekaMine](http://jdurbin.github.io/wekaMine/) encapsulates all of this for you and gives much more [SciKit-learn](cikit-learn.org) style of interfaces to Weka.  
