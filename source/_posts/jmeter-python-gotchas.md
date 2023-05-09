---
title: JMeter & python gotchas
date: 2019-03-06 12:52:37
photos: 
	- /img/posts/article-pict-header.jpg
tags: [jmeter, python]
categories:
 - [jmeter]
 - [python]
---

Using python is easy with JMeter, which doesn't mean one should not be careful with how it's implemented. Here you will find out about examples, how to deal with it.

<!--more-->

### Datetime Objects

There are some things to watch out for when using  {% link jython https://www.jython.org/ %}/python in JSR223 code in Jmeter.

As Jython is a java implementation of Python you may come across some "Type" bugs.

Let's look at the datetime object.

If we wanted to create 2 variables for our script, 1 being the current datetime and the other being 10 days from now.
This is a common case when scripting for airline flights.

In native Python we can just do something like:

```python
import datetime

date = datetime.datetime.now()
date_plus_10_days = date + datetime.timedelta(days=10)
print("current_date is %s, date_plus_10_days is %s" % (date, date_plus_10_days))

current_date is 2019-03-06 13:06:30.941109, date_plus_10_days is 2019-03-16 13:06:30.941109
```

However when using this in jmeter JSR223 code we get the following error.

![img.01](jython_gotchya_datetime_error.png)

This is because of a "TypeError". When Jython assigns a datetime object to a variable, it transforms it to a java.sql.Timestamp object.

![img.02](jython_datetime_java_type.png)

So, in order to use datetime objects in jython, you need to use them immediately and not assign them to a variable.

![img.03]( jython_datetime_oneliner_type.png)

This makes the code less eloquent, but it looks like this "bug/feature" won't be fixed, so you will have to use this workaround.

![img.04](jython_datetime_oneline_code_fix.png)


