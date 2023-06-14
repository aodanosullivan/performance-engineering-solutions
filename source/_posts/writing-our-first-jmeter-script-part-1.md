---
title: Writing our first jmeter script - part 1
date: 2018-05-20 14:55:47
author: Aodan
photos: 
	- /img/posts/jmeter.png
tags: [jmeter]
categories: 
 - [jmeter]
---

For this exercise we are going to write a jmeter script against a sample website. We are going to use demo of prestashop, free e-commerce software to set up our own local website.

<!--more-->

### Setting up environment.

You can find mentioned demo here: {% link prestashop‘s http://demo.prestashop.com/en/?view=front %} free e-commerce software to set up our own local website.

If you haven’t already, {% post_link jmeter-basic-installation "install jmeter" %} on your local machine.

{% blockquote %}
Note: Please don't execute high traffic against public websites without their consent first.
{% endblockquote %}

To install the site locally, we are going to use {% link docker https://docs.docker.com/get-started/ %}. If you are unfamiliar with docker you can click the link above for a tutorial. Docker provides a way to run applications securely isolated in a container, packaged with all its dependencies and libraries.

You can find instructions on how to download the prestashop’s docker container {% link here https://hub.docker.com/r/prestashop/prestashop/ %}.

First thing we need to do is install the DB.

{% blockquote %}
    \# docker run -ti \-\-name training-mysql -e MYSQL_ROOT_PASSWORD=your-password -d mysql:5.7
{% endblockquote %}

*Note: docker needs to execute with root priviledges. You can execute this command as root but it is recommended to use “sudo” or add your user to the docker group. When using docker it is better to only use images from trusted resources.*

Once you execute this command, if you don’t have the docker image locally, it will pull it down from the docker repository.

{% blockquote %}
Unable to find image 'mysql:latest' locally
latest: Pulling from library/mysql

85b1f47fba49: Pull complete 
27dc53f13a11: Pull complete 
095c8ae4182d: Pull complete 
0972f6b9a7de: Pull complete 
1b199048e1da: Pull complete 
159de3cf101e: Pull complete 
963d934c2fcd: Pull complete 
f4b66a97a0d0: Pull complete 
f34057997f40: Pull complete 
ca1db9a06aa4: Pull complete 
0f913cb2cc0c: Pull complete 
Digest: sha256:bfb22e93ee87c6aab6c1c9a4e7cdc68e9cb9b64920f28fa289f9ffae9fe8e173
Status: Downloaded newer image for mysql:latest
725e21f338112a1e55184dd80ad28875d56b43935c80a502828f950fc16b0a2e
{% endblockquote %}

Now we can pull down our prestashop image and point it to the mysql container we just ran.


{% blockquote %}
    \# docker run -ti \-\-name training-prestashop \-\-link training-mysql -e DB_SERVER=training-mysql -p 8080:80 -d prestashop/prestashop
{% endblockquote %}

Again is will pull down the image from the server if it can’t find it locally. The -p option tells docker to map port 8080 on our local machine to port 80 inside the container.

To check if the containers are running, you can do a ‘docker ps’ command.

{% blockquote %}
    \# sudo docker ps
    CONTAINER ID        IMAGE                   COMMAND                  CREATED              STATUS              PORTS                  NAMES
    26923ecb2431        prestashop/prestashop   "docker-php-entrypoin"   About a minute ago   Up About a minute   0.0.0.0:8080->80/tcp   training-prestashop
    725e21f33811        mysql                   "docker-entrypoint.sh"   7 minutes ago        Up 7 minutes        3306/tcp               training-mysql
{% endblockquote %}

If you ever want to stop and restart these containers it is as simple as doing:

{% blockquote %}
    \# docker stop training-prestashop 
    \# docker stop training-mysql
    \# docker start training-mysql
    \# docker start training-prestashop
{% endblockquote %}


Once you go to http://localhost:8080/ you will be presented with the configuration wizard to set up your “shop”.


![img.01](prestashop_start-768x450.png)

Follow the wizard.

![img.02](store-info-768x447.png)

When configuring the database it will default to 127.0.0.1. As we are running our DB in a container, we need to point this to the ipaddress of the container.

You can get the ipaddress of your container by running “docker inspect” command. (If you are unfamiliar with linux commands, I am filtering the last IPAddress entry from the output.). Your IPAddress may differ

{% blockquote %}
    \#docker inspect training-mysql | grep IPAddress| tail -1
                    "IPAddress": "172.17.0.2",
{% endblockquote %}

Enter in your information and Click on the “Test your Database Now” button and create your DB.

![img.03](prestashop_db_created-768x389.png)

Click Next and follow the on-screen instructions to complete the installation of your test site. Take note of your login/password information. You may need this in the future to increase your “Stock” after running a performance test.

*Note: If you are using this behind a proxy you have to configure your prestashop container to use proxy.*

In order to start using the site, we must first enter the docker container and delete the “install’ folder.

{% blockquote %}
    \# docker exec -it training-prestashop /bin/bash
    root@26923ecb2431:/var/www/html/# ls -lart
    total 740
    -rw-r--r--   1 www-data www-data   1168 Oct 14 16:24 init.php
    -rw-r--r--   1 www-data www-data   1082 Oct 14 16:24 index.php
    -rw-r--r--   1 www-data www-data   4733 Oct 14 16:24 images.inc.php
    -rw-r--r--   1 www-data www-data   2454 Oct 14 16:24 error500.html
    -rw-r--r--   1 www-data www-data 250394 Oct 14 16:24 composer.lock
    -rw-r--r--   1 www-data www-data 357285 Oct 14 16:24 LICENSES
    -rw-r--r--   1 www-data www-data   4941 Oct 14 16:24 INSTALL.txt
    drwxr-xr-x  10 root     root       4096 Oct 16 10:51 ..
    drwxr-xr-x  46 www-data www-data   4096 Oct 16 10:51 classes
    drwxr-xr-x  10 www-data www-data   4096 Oct 16 10:51 app
    drwxr-xr-x   6 www-data www-data   4096 Oct 16 10:51 themes
    drwxr-xr-x   6 www-data www-data   4096 Oct 16 10:51 docs
    drwxr-xr-x   2 www-data www-data   4096 Oct 16 10:51 bin
    drwxr-xr-x  12 www-data www-data   4096 Oct 16 10:51 js
    drwxr-xr-x  18 www-data www-data   4096 Oct 16 10:51 install
    drwxr-xr-x  14 www-data www-data   4096 Oct 16 10:51 cache
    drwxr-xr-x   8 www-data www-data   4096 Oct 16 10:51 override
    drwxr-xr-x   8 www-data www-data   4096 Oct 16 10:51 tools
    drwxr-xr-x  16 www-data www-data   4096 Oct 16 10:51 admin
    drwxr-xr-x  68 www-data www-data   4096 Oct 16 10:51 vendor
    drwxr-xr-x   6 www-data www-data   4096 Oct 16 10:51 controllers
    drwxr-xr-x   2 www-data www-data   4096 Oct 16 10:51 webservice
    drwxr-xr-x   2 www-data www-data   4096 Oct 16 10:51 pdf
    drwxr-xr-x   8 www-data www-data   4096 Oct 16 10:51 src
    drwxr-xr-x   2 www-data www-data   4096 Oct 16 10:51 localization
    drwxr-xr-x  50 www-data www-data   4096 Oct 16 11:18 .
    -rw-r--r--   1 www-data www-data   2670 Oct 16 11:18 robots.txt
    -rw-rw-rw-   1 www-data www-data   2773 Oct 16 11:18 .htaccess
    drwxr-xr-x  34 www-data www-data   4096 Oct 16 11:20 img
    drwxr-xr-x 114 www-data www-data   4096 Oct 16 11:20 modules
    drwxr-xr-x   6 www-data www-data   4096 Oct 16 11:20 translations
    drwxr-xr-x   2 www-data www-data   4096 Oct 16 11:20 upload
    drwxr-xr-x   4 www-data www-data   4096 Oct 16 11:20 mails
    drwxr-xr-x   2 www-data www-data   4096 Oct 16 11:20 download
    drwxr-xr-x   6 www-data www-data   4096 Oct 16 11:20 config
    root@26923ecb2431:/var/www/html/# rm -Rf install
    root@26923ecb2431:/var/www/html/# 
{% endblockquote %}

![img.04](prestashop_main-768x386.png)

Author: {% link "Aodan" https://performance-engineering-solutions.com/team/index.html#Aodan %}
