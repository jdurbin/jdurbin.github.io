---
layout: post
title: "Using Groovlets in jQuery tutorial"
modified:
categories: blog
excerpt:
tags: [AJAX,groovy,jQuery]
image:
  feature:
date: 2010-7-10
---

I've been learning a little about jQuery by going through the excellent [jQuery tutorial](http://docs.jquery.com/Tutorials:Getting_Started_with_jQuery) over at the [jQuery Docs](http://docs.jquery.com/Main_Page) page. The examples use `PHP` for server side code, but since I'm more familiar with Servlets/[Groovlets](http://www.groovy-lang.org/servlet.html), I decided do the server side code as a Groovlet.


##Setup
After [installing Tomcat](https://tomcat.apache.org/tomcat-7.0-doc/appdev/installation.html), I created a subdirectory under webapps called sandbox. In sandbox, I created a `sandbox/WEB-INF` directory with a lib subdirectory. I copied all of `/usr/local/groovy/lib/` to `sandbox/WEB-INF/lib/`, though only some of it is needed it was just easier to copy everything.  I then put the following `web.xml` file under `sandbox/WEB-INF/`

###web.xml
{% highlight xml %}
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
  version="2.5">
  <display-name>Groove</display-name>
    <servlet>
      <servlet-name>GroovyServlet</servlet-name>
      <servlet-class>groovy.servlet.GroovyServlet</servlet-class>
    </servlet>
    <servlet>
        <servlet-name>GroovyTemplate</servlet-name>
        <servlet-class>groovy.servlet.TemplateServlet</servlet-class>    
  </servlet>
    <servlet-mapping>
        <servlet-name>GroovyServlet</servlet-name>
        <url-pattern>*.groovy</url-pattern>
    </servlet-mapping>
    <servlet-mapping>
        <servlet-name>GroovyTemplate</servlet-name>
        <url-pattern>*.gsp</url-pattern>
    </servlet-mapping>
</web-app>
{% endhighlight %}

This `web.xml` will tell Tomcat to use `groovy.servlet.GroovyServlet` to handle `*.groovy` files. From then on, any .groovy files you put under /sandbox (or any subdirectories thereof) will be compiled as a Groovlet by Tomcat. 

I then downloaded the jquery-starterkit from the jquery site and unpacked it into sandbox/jqstarterkit. In jqstarterkit are a number of files, custom.js, rate.php, screen.css, and starterkit.html. You will need to [download jquery](http://code.jquery.com/jquery-1.4.2.min.js) itself into /sandbox/jqstarterkit/ and rename it to `/sandbox/jqstarterkit/jquery.min.js`. Then go through the tutorial, editing custom.js to include the custom bits of jQuery javascript in the tutorial, seeing the results in:

`http://localhost:8080/sandbox/jqstarterkit/starterkit.html`

### rate.php -> rate.groovy

When you get to the part about `rate.php`, simply replace rate.php with `rate.groovy` in the `custom.js` script, like so:

{% highlight javascript %}
jQuery(document).ready(function(){
	// generate markup  $("#rating").append(" Please rate: ");
	for ( var i = 1; i <= 10; i++ )
		$("#rating").append("<a href='#'>" + i + "</a> ");

	// add markup to container and apply click handlers to anchors  
	$("#rating a").click(function(e){
		// stop normal link click
		e.preventDefault();
		// send request
		$.post("rate.groovy", {rating: $(this).html()}, function(xml) {
			// format and output result
			$("#rating").html(
				"Thanks for rating, current average: " +
				$("average", xml).text() +
				", number of votes: " +$("count", xml).text()
			);
		});
	});
});
{% endhighlight %}

### rate.groovy
{% highlight groovy %}

// Simple AJAX groovlet... takes a rating from a jQuery $.post()
// saves it to a file, computes the running average, then 
// returns a status line in XML format...
rating = request.getParameter('rating')
f = new File('sandbox/jqstarterkit/ratings.dat')
f << "$rating\n"

// Now, read in the file and compute number of ratings
// and average rating...sum = 0;
count = 0;
f.eachLine{
  sum+= (it as int)
  count++
}
avg = (double)sum/(double)count

// Report result back to jQuery
// In Groovlet, html is bound to new MarkupBuilder(out) by default
// so the following bit of markup substitutes for this string:
// out<<"""
//  <ratings>
//    <average>$avg</average>
//    <count>$count</count>
//  </ratings>"""
// 
// out itself is bound to 
response.getOutputStream()html.ratings(){
  average("$avg")
  count("$count")
}
{% endhighlight %}


When a .groovy file is interpreted as a Groovlet, several variables are pre-bound. `request`, for example, is bound to `ServeletRequest`. See the full list of bindings in the Groovlet docs. It took me a bit of trial and error to get all of this right and working. Hopefully this boilerplate will help someone else get up and going even quicker.


