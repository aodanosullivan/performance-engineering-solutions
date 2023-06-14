---
title: Using python in JMeter
date: 2018-05-14 14:39:51
author: Aodan
photos: 
	- /img/posts/jmeter.png
tags: [jmeter, python]
categories:
 - [jmeter]
 - [python]
---

JMeter has many features. Among them there is possibility to implement extra functionality in languages like java, groovy and python. In this post, closer approach for python is going to be discussed.

<!--more-->

### Setup

Installing Python for jmeter is easy. First download the jython.jar from {% link here http://www.jython.org/downloads.html %}.

If you use the layout suggested in my {% post_link jmeter-basic-installation "installation guide" %}, place the jython.jar in your {project_home}/lib directory and start jmeter.

If you use the default installation, place the jython.jar in your {jmeter_home}/lib directory and start jmeter.

Once you start up your jmeter, in your thread group you now have a python & jython option for your JSR223 PreProcessor, Sampler and PostProcessor items.

![img.01](python_jmeter-1024x558.png)


As a general rule of thumb I use, if your python script is over a 10 lines of code, it is better to move the code out of jmeter and into its own script. You can store this in your {project_folder}/scripts/lib directory and browse/link to it in the “Script file” section in the JSR223 sampler.

{% blockquote %}
Note: Remember to click "Cache compiled script" (or give it an unique cache file name if using previous versions of jmeter) as the last thing we want is for our loaddriver to be the bottleneck in our tests.
{% endblockquote %}

This is just a short example. Over the course of this blog we will use more python in our jmeter scripts.

Author: {% link "Aodan" https://performance-engineering-solutions.com/team/index.html#Aodan %}
