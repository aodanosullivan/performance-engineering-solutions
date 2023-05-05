---
title: Writing our first jmeter script - part 2
date: 2018-11-27 21:05:54
photos: 
	- /img/posts/article-pict-header.jpg
tags: [jmeter]
categories:
 - [jmeter]
---

### Using the Recording Controller.

Now that we have our {% post_link writing-our-first-jmeter-script-part-1 "local website" %} installed, we will record traffic against this using jmeter.

First create a new script and add a basic thread group. (We can change this easily later)
To record, it is not necessary to add a thread group but it makes things easier going forward.

<!--more-->

![img.01](jmeter-simple-thread-group.png)

Next add the HTTP(S) Test Script Recorder by right-clicking on Test Plan -> Add -> Non-Test Elements -> HTTP(S) Test Script Recorder


![img.02](https-test-script-recorder.png)

