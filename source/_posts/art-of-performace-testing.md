---
title: The Art & Science of Performance Engineering
date: 2018-05-13 15:07:44
photos: 
	- /img/posts/article-pict-header.jpg
---

I've heard lots of people talk about the “Art of Performance Engineering”. At the start of your career it may seem like an art but it should evolve into a science which will allow you to create even better art. Some people believe that the best art is only obtained by people with the expert knowledge of the science behind the components of the art. One of the  goals of this blog is to help junior Performance Engineers understand the science in order to create their own art.

<!--more-->

### Performance engineering

I read a quote on the net (can't find the source now) that stated “There is no such thing as a Junior Performance  Engineer”. This is an interesting quote and can be interpreted several ways.

As a consultant, this is more true. A Performance Engineering consultant should be experienced, have the knowledge to identify issues and be able to pinpoint the cause of these issues. They should also be able to communicate the results at the right detail to particular audiences. No one likes to hear that their baby is ugly but this is often the message that a Performance Engineer has to deliver to development teams. The way you deliver this message is key to getting the development teams working together to reach the same goal.

As an employee, this is less true but also valid. Good Performance Engineers are  DevOps/Automation Engineers, statisticians, testers, have good knowledge of Linux and JVM internals and an unending thirst for knowledge. Of course it is very rare to find candidates with all these qualities so we spend a good proportion of time on training. Only the most experienced Performance Engineers have all these attributes. Obviously though, you have to start somewhere. So where to start?

* A scripting language is essential. My choice is {% link Python https://www.python.org/ %} and I’ll probably show some examples in this blog throughout the series, but almost any will do. However, if you use VB just stop reading and examine your life choices.
* I recommend using {% link Jmeter http://jmeter.apache.org/ %}  as a loaddriver if you don't have to write your own custom one. Jmeter is a open source loaddriver and there are a lot of plugins available. This blog will contain several articles/tutorials on Jmeter.
* In-depth Linux knowledge. Regardless of what OS you are working on at the moment, the likely-hood is that at some point in your career you will be using or testing applications running on Linux.
* {% link “Systems Performance: Enterprise and the cloud” http://www.brendangregg.com/sysperfbook.html %}  by Brendan Gregg. This book has been called the bible of Performance Engineering but like the bible, can be tricky to understand at first. I recommend getting a good understanding of Linux and Performance Engineering before tackling this book. The author was a Performance Engineer in Sun/Oracle before becoming a Senior Performance Architect at Netflix. Did you ever see the youtube clip of someone {% link "screaming in a datacenter" https://www.youtube.com/watch?v=tDacjrSCeq4 %}? That was Brendan Gregg.

