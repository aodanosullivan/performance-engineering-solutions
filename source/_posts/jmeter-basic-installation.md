---
title: JMeter basic installation
date: 2018-05-14 14:52:22
author: Aodan
photos: 
	- /img/posts/jmeter.png
tags: [jmeter]
categories:
 - [jmeter]
---

As with most of the posts here, I will be using linux. In a few simple steps you find out how to get JMeter to work on you local machine.

<!--more-->

### Downloading & installation

Get you installation package from the jmeter site.  I recommend using version 3.0+. You will see why in later posts. Download the binaries in either tar.gz or zip format and verify the integrity. Then extract it to a dedicated location for your loaddrivers. You may have multiple versions of jmeter and upgrading to new versions is easier.

For tar files

{% codeblock lang:bash %}
    tar xvfz ~/Downloads/apache-jmeter-3.1.tgz --directory /path/to/directory
{% endcodeblock %}

For zip files
{% codeblock lang:bash %}
    unzip ~/Downloads/apache-jmeter-3.1.zip -d /path/to/directory
{% endcodeblock %}

Done.!! We will be leaving these directories untouched.

{% codeblock lang:bash %}
    ~/training/loaddrivers$ ls
    apache-jmeter-3.0  apache-jmeter-3.1
{% endcodeblock %}


### Project Folder Structure
I recommend the following project folder structure.

{% blockquote %}
    {wherever}/projects/{application}/
        ./lib ## location for additional supporting libraries that would usually go in {jmeter_home}/lib
        ./lib/ext ## location for additional external plugins that would usually go in {jmeter}/lib/ext
        ./properties ## location for your jmeter property files (if used)
        ./scripts ## location where you store your jmeter scripts
        ./scripts/lib ## location where you can store any additional custom helper scripts 
        ./scripts/data ## location to store any payloads you might need
        ./scenarios ## location to store any scenarios you create
        ./logs ## location where you store the test logs
        ./jmeter ## symbolic link to the jmeter installation you want to use
        ./java ## symbolic link to the java installation you want to use
        ./run_jmeter.sh ## how you will run jmeter
{% endblockquote %}


This directory structure allows you a lot of flexibility and provides structure for your project. Switching the version of jmeter is as simple and changing the symlink (if plugins are compatible) . The same goes for your java version. The run_jmeter.sh script will pass all your arguments to the {jmeter_home}/bin/jmeter.sh script as well as setup some JMETER variables. 

You might ask why create this extra script just to call the jmeter.sh script. This allows you to separate your development environment and your production environment. In your project folder you only have the plugins/libraries/scripts etc required for your project. A common jmeter installation can be used for everyone.

Sample run_jmeter.sh script

{% codeblock lang:bash %}
    #!/usr/bin/env bash
    
    ## Get the directory where this script is located
    directory="$( cd "$( dirname $0)" && pwd )"
    ## Build the ${plugin} variable
    for files in `find ${directory}/lib/ext -maxdepth 1 -type f`; do
    plugins="${plugins};${files}"
    done
    ## Build the libraries variable
    for files in `find ${directory}/lib -maxdepth 1 -type f`; do
    libraries="${libraries}:${files}"
    done
    ## Remove the first ; or :
    plugins=`echo ${plugins} | sed -e 's/^;//'`
    libraries=`echo ${libraries} | sed -e 's/^://'`
    
    ## Build the jmeter options for plugins and libraries
    search_paths=`echo "-Jsearch_paths=${plugins}"`
    class_path=`echo "-Juser.classpath=${libraries}"`
    
    ## Set your JAVA location by adding it to the $PATH variable
    JAVA_HOME="${directory}/java"
    export PATH="${JAVA_HOME}/bin:${PATH}"
    ## JVM_ARGS & JMETER_OPTS etc can be placed here. Just make sure that you
    ## add them to the command at the end
    
    ## Start jmeter
    ## The "$@" will pass any arguments from the command line to the jmeter.sh script
    ${directory}/jmeter/bin/jmeter.sh ${search_paths} ${class_path} \
    -j ${directory}/logs/jmeter.log ${any_other_variables} "$@"

{% endcodeblock %}

Author: {% link "Aodan" https://performance-engineering-solutions.com/team/index.html#Aodan %}
