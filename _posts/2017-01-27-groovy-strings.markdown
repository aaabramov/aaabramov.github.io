---
layout: post
title:  "Groovy strings"
date:   2017-01-16 22:21:15 +0200
categories: groovy
---

Why Groovy strings are so awesome?
---

At first, let's see which types of string are there in Groovy.

The first type is a plains string. It is a literal surrounded with single quote.

For example:

{% highlight groovy %}
def name = 'John'
println name
{% endhighlight %}
>\> John

Nothing special about it. Just a boring Java string.

But the power is actually in Groovy string, a.k.a. [GString][GString docs]. It is a literal that is surrounded with double quotes.

Let's start from an example.

~~~groovy
def name = 'John'
println "Hello, $name"
~~~
>\> Hello, John

We can avoid concatenation like `"Hello, " + name` in code thankfully to the GStrings.
With such syntax you can substitute not only variable values but also whole expressions:

~~~groovy
def name = 'John'
println "Hello, ${name.toUpperCase()}"
~~~
>\> Hello, John  
>\> 2 * 2 = 4

Be sure that you put curly braces around the expression. Because if you want, you will get result like:

~~~groovy
def name = 'John'
println "Hello, $name.toUpperCase()"
~~~
>\> Caught: groovy.lang.MissingPropertyException: No such property: toUpperCase for class: java.lang.String
groovy.lang.MissingPropertyException: No such property: toUpperCase for class: java.lang.String

For escaping special symbols is GStrings you can put `\` characher before them:

~~~groovy
println "Result of \${20 + 5} = ${20 + 5}"
~~~
>\> Result of ${20 + 5} = 25

[GString docs]: http://docs.groovy-lang.org/latest/html/api/groovy/lang/GString.html