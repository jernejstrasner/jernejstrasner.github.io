---
layout: post
title: "A simple ZSH todo function"
date: 2014-01-22 19:00:00
tags: [zsh]
---

I got tired of all todo apps and needed a simpler way of handling my everyday tasks.
My OS X desktop was mostly empty, I was only keeping files that reminded me of tasks.
So I though, why not use my desktop as my todo list? I already spend a lot of time in
the terminal so a simple ZSH todo function solved all my todo problems.

{% highlight bash %}
# Simple todo function
function todo() {
	if (( $# == 2 )); then
		echo $1"\n\n"$2 > ~/Desktop/$1.txt
	elif (( $# == 1 )); then
		echo $1 > ~/Desktop/$1.txt
	else
		echo "Not enough arguments!"
	fi
}
{% endhighlight %}

Bonus: If you're using Alfred on OSX, you can use the workflow below. Just type "todo" in Alfred's command window and make sure you escape or quote the arguments if they contain spaces.

[TODO Alfred Workflow]({{ site.url }}/files/zsh_todo.alfredworkflow)
