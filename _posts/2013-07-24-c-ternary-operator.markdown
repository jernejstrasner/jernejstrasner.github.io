---
layout: post
title:  "The ternary operator in C"
date:   2013-07-24 12:58:00
categories: c tip gcc clang
---

Most programmers are familiar with the ternary operator [?:](http://en.wikipedia.org/wiki/%3F:#C) that is available in C and most other languages.
But what some people may not know is that you can also use this operator with only two arguments.

Instead of writing

{% highlight c %}
int y = 23;
int x = (y ? y : 34);
{% endhighlight %}

you can do this

{% highlight c %}
int y = 23;
int x = (y ?: 34);
{% endhighlight %}

The other thing some of you may not know is that when defining a const you can't use an if statement to define it based on a condition, but you can do this:

{% highlight c %}
int y = 23;
const int x = (y ?: 34);
{% endhighlight %}

That's it for the first post!
