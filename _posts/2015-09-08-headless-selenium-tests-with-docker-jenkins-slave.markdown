---
layout: post
title: "Headless Selenium Test With Docker Jenkins Slave"
date: 2015-09-07T08:15:57-06:00
---

In an earlier blog post, I wrote about building Docker based jenkins slaves. I used that to automate deployment of our system and clients from Jenkins. With that working, I wanted to apply the same to running some of our UI automation. We're currently using a series of Windows virtual machines as remote slaves and Sikuli based tests. The downside to these tests is that they are based on image capture and actually driving a browser, so we can only one run at a time per slave and they can fail if resolution of the VM is changed or something is launched that is on top of the browser. One fix for this would be to use something like selenium instead. I implemented a subset of the tests using selenium and I've been able to run these on my laptop using the Firefox browser just fine. 

With the tests running with selenium, I wondered if we could run them headlessly via the Docker slave mechanism. To do that, I needed a new slave image. Starting with the evarga/jenkins-slave base image, I installed maven, git, firefox and xfvb. I also generated an ssh key, registered it with our git instance and then added it to the container.   

~~~~
FROM evarga/jenkins-slave:latest

ENV DEBIAN_FRONTEND noninteractive
ENV DEBCONF_NONINTERACTIVE_SEEN true

#===================
# Timezone settings
# Possible alternative: https://github.com/docker/docker/issues/3359#issuecomment-32150214
#===================
ENV TZ "US/Mountain"
RUN echo "US/Mountain" | sudo tee /etc/timezone \
  && dpkg-reconfigure --frontend noninteractive tzdata

#==============
# Xvfb, firefox, maven, git
#==============
RUN apt-get update -qqy \
  && apt-get -qqy install \
    xvfb firefox maven git \
  && rm -rf /var/lib/apt/lists/*

#============================
# Some configuration options, same size as MacBook display
#============================
ENV SCREEN_WIDTH 1680
ENV SCREEN_HEIGHT 1050
ENV SCREEN_DEPTH 24
ENV DISPLAY :99.0

COPY id_rsa /home/jenkins/.ssh/
COPY id_rsa.pub /home/jenkins/.ssh/
RUN chown jenkins /home/jenkins/.ssh/
RUN chmod 600 /home/jenkins/.ssh/id_rsa
USER jenkins
RUN ssh-keyscan qeconfig.wp.fsi >> /home/jenkins/.ssh/known_hosts
RUN chmod 600 /home/jenkins/.ssh/id_rsa.pub
RUN chmod 600 /home/jenkins/.ssh/known_hosts
USER root
~~~~

When all is said and done, that's a fairly large image. Again, starting from scratch might lead to a smaller image.

~~~~
jeremy@euclid [ ~/ui-test ]$ sudo docker images
REPOSITORY             TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ui-test.slave          latest              f06dd6a0ba85        35 seconds ago      832.2 MB
~~~~

With the image created, I followed the same process of adding a new template to the Docker cloud defined in Jenkins. Then created a new job. 

![Job config]({{site.url}}/images/docker-label.png)
![Shell commands]({{site.url}}/images/shell-commands.png)
