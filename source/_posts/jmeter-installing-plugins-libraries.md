---
title: jmeter-installing-plugins-libraries
date: 2018-05-14 19:06:47
tags: [jmeter]
categories: 
 - [jmeter]
---

## Plugins
As mentioned in a previous post, using the directory structure provided and the run_jmeter.sh script, we can separate our jmeter installation from our plugins/libraries. To do this, all you have to do is copy your plugins to the {project}/lib/ext/ folder.

{% link "jmeter-plugins.org" https://jmeter-plugins.org/ %}  has a wide variety of plugins available (or you could write your own). You can install these plugins individually or use the plugin manager. 2 plugins I highly recommend are the {% link "Throughput Shaping Timer" https://jmeter-plugins.org/wiki/ThroughputShapingTimer/ %}  and the {% link "Ultimate Thread Group" https://jmeter-plugins.org/wiki/ConcurrencyThreadGroup/ %} . These plugins allow you to shape your test traffic the way that you want.

To install these, download from the links above and place in your {project}/lib/ext folder. Once you start jmeter these will be available the Test plan menu.

{% asset_img UltimateThreadGroup_menu.png %}  


{% asset_img ThroughputShapingTimer_menu.png %}  

Both plugins use steps to shape the number of threads or the requests per second but they differ slightly in their setup.

### Ultimate Thread Group

This plugin takes 5 parameters.

1. Start Threads Count (number of threads to start)
2. Initial Delay (use If you need to add more threads at a later time)
3. Startup time (use If you need to ramp up threads at a slower rate)
4. Hold Load for (duration)
5. Shutdown Time (use if you want to slowly kill the threads instead of an abrupt stop)

There is a way to control these settings from user.properties, jmeter.properties or from the command line. When we design our scripts, we want them as flexible as possible without having to edit them in jmeter. For more info on the Ultimate Thread Group see {% link here https://jmeter-plugins.org/wiki/UltimateThreadGroup/?utm_source=jmeter&utm_medium=helplink&utm_campaign=UltimateThreadGroup %}.

{% blockquote %}

Note:
Threads in jmeter can be considered as your virtual users. If your application stores users sessions and if your jmeter script does anything on initialization of a thread, exercise caution when designing your thread profile for your test. See example below:

{% endblockquote %}

### Scenario:
You want to run a test for 1 hour. The first 30 minutes will have 100 threads/users and the second 30 minutes will have 200 threads/users. In the screenshots below, both have the same profile, but with one very important difference. In the wrong way, what happens is at the start of the test it creates 100 threads/users. Let’s say all these users sign into your application with a unique id and your application caches the session/performs some functionality. After 30 minutes, jmeter kills all these 100 threads/users and starts up 200 threads/users, again you sign in with your unique id. So, instead of your application storing 200 unique user sessions, it is now storing 300 (until the application cleans it up).

To better reflect an increase in users, you should take the approach of the right way screenshot. Instead of having 100 users for 30 minutes, you set 100 users for the hour. Then after 30 minutes add another 100 users.


#### Wrong way

{% asset_img UltimateThreadGroup_wrong.png %}

#### Right way

{% asset_img UltimateThreadGroup_right.png %}


### Throughput Shaping Timer
Like the Ultimate Thread Group, you are also able to control the {% link "Throughput Shaping Timer" https://jmeter-plugins.org/wiki/ThroughputShapingTimer/?utm_source=jmeter&utm_medium=helplink&utm_campaign=ThroughputShapingTimer %} from user.properties, jmeter.properties and from the command line.

Unlike the Ultimate Thread Group, this has only 3 parameters. After each step is complete, it moves on to the next step. There is no concept of *Initial Delay*. In our previous example, if we wanted to emulate a user executing a transaction every 2 seconds, we can have the following throughput shaping timer.



{% asset_img ThroughputShapingTimer.png %}

{% blockquote %}

Note:
Even if you are not worried about user sessions you need to ensure that you have the enough threads to handle your throughput. Eg. if the max response time you will ever get is 2000ms, then to run 50 RPS, you will need to have at least 100 threads.

{% endblockquote %}

{% link "jmeter-plugins.org" https://jmeter-plugins.org/ %} now has a plugin manager. Unfortunately, you can't specify where the plugin gets installed and it defaults to your jmeter installation. If you don't have multiple projects and don’t need to keep jmeter/external files separate you can use this to manage your plugins.

## Libraries

By installing external libraries, we can use them in jmeter. Next I will show you how to use {% post_link using-python-in-jmeter "Python in jmeter" %}.
