---
layout: post
title: "Creating a Docker Jenkins Slave Running on a VMware Photon VM"
date: 2015-09-03T20:13:27-06:00
---

Let's setup a Docker based Jenkins slave using VMware Photon (currently using Tech Preview #2) as the docker host. 

This assumes you have Jenkins installed.

## Install Jenkins Plugin(s)

First, let's install the Jenkins [Docker Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Docker+Plugin), plus all of it's dependencies (if you don't already have them):
 * [Durable Task](https://wiki.jenkins-ci.org/display/JENKINS/Durable+Task+Plugin)
 * [Token Macro](https://wiki.jenkins-ci.org/display/JENKINS/Token+Macro+Plugin)
 * [SSH Slaves](https://wiki.jenkins-ci.org/display/JENKINS/SSH+Slaves+plugin)
 * [SSH Credentials](https://wiki.jenkins-ci.org/display/JENKINS/SSH+Credentials+Plugin)
 * [Credentials](https://wiki.jenkins-ci.org/display/JENKINS/Credentials+Plugin)


## Install Photon

Next, let's grab the [Photon 1.0 TP2 ISO](https://bintray.com/vmware/photon/iso/1.0TP2/view), create a new virtual machine and use the Photon ISO to install. I did the full install, but the minimal would probably work as well. My full install took 70 seconds. 

Once installed, open the console to the VM using the vSphere Web Client and log into the VM. By default, a Photon installation doesn't allow root to SSH into the VM. So let's create a user to do the work we need to do.

First, let's enable sudo access by using visudo to edit the sudoers file. 
Find this block in the sudoers file:

~~~~
## Uncomment to allow members of group wheel to execute any command
#%wheel ALL=(ALL) ALL
~~~~

And uncomment the second line (#%wheel ALL=(ALL) ALL.

Next, let's add a user with useradd: 

~~~~
root [ / ]$ useradd --create-home --shell=/bin/bash jeremy
~~~~

Next, add it to the wheel group:

~~~~
root [ / ]$  usermod -aG wheel jeremy
~~~~

Now set it's password:

~~~~
root [ / ]$ passwd jeremy
~~~~

## Docker Config

Now you should be able to login via your favorite means of sshing. Now let's move on to setting up the VM as a Docker based Jenkins slave.

Note that by default, a user created above has very limited privileges. So most of the following commands are done via sudo.

First, let's enable docker.  :
~~~~
jeremy@euclid [ ~ ]$ sudo systemctl start docker
jeremy@euclid [ ~ ]$ sudo systemctl enable docker
~~~~

Now let's enable remote management of Docker so Jenkins can connect. To do this, we need to modify  /etc/systemd/system/multi-user.target.wants/docker.service and enable remote access to the Docker api. This is normally turned off. Really, this is the most Photon specific part of this entire gist. You'll see the location of that file when you execute the systemctl enable docker command above. Find the ExecStart line and edit it like so:

~~~~
ExecStart=/bin/docker -d -s overlay -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375
~~~~

Now, reload the systemctl daemon and restat the Docker daemon:

~~~~
jeremy@euclid [ ~ ]$ sudo systemctl daemon-reload
jeremy@euclid [ ~ ]$ sudo systemctl restart docker
~~~~

## Configure Jenkins and Docker Slave

Now, let's grab the ready made Jenkins slave image:

~~~~
jeremy@euclid [ ~ ]$ sudo docker pull evarga/jenkins-slave
~~~~

Now you should have the slave image:
~~~~
jeremy@euclid [ ~ ]$ sudo docker images
REPOSITORY             TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
evarga/jenkins-slave   latest              8880612971b0        8 months ago        610.7 MB
~~~~

I'm going to use this to deploy our application which is currently packaged as an OVA, so I will use the jenkins-slave image as the base for my new image. Then I will install ovftool along with python because we already have some deploy automation with it. Here is my new Dockerfile:  

~~~~
FROM evarga/jenkins-slave:latest

RUN mkdir -p /opt/vmware
COPY pycaaso.py /opt/vmware/
COPY scale-weekly.conf /opt/vmware/
RUN apt-get install python -y
RUN mkdir -p /opt/ovftool
ADD VMware-ovftool-4.1.0-2459827-lin.x86_64.bundle /opt/ovftool/
RUN yes | /bin/bash /opt/ovftool/VMware-ovftool-4.1.0-2459827-lin.x86_64.bundle --required --console \
    && rm -f /opt/ovftool/VMware-ovftool-4.1.0-2459827-lin.x86_64.bundle
~~~~
Now, rebuild it and tag it so we can use it from Jenkins:

~~~~
jeremy@euclid [ ~ ]$ sudo docker build -t jenkins-slave2 .
~~~~

Now we can go configure Jenkins. The Docker plugin makes your docker slaves available as a remote cloud, not as remote nodes. So when you configure, you don't actually use the normal add remote node workflow that you'd do if you were adding a traditional Windows or Linux remote slave. To configure the Docker connectivity, first login to Jenkins and click Manage Jenkins:

![mange jenkins](https://gist.github.com/jrrickard/114b8c35b1d5306ff3e0/raw/d7d915e03c4835e0d99a27594a65559ed907f72c/manage-jenkins.png)

Then click Configure System: 

![configure system](https://gist.github.com/jrrickard/114b8c35b1d5306ff3e0/raw/9ecf828f23dd1133448a104e0dca0297dff64e6c/configure-system.png)

Then scroll down to Cloud and Click Add New Cloud, then Select Docker.

![add new cloud](https://gist.github.com/jrrickard/114b8c35b1d5306ff3e0/raw/d7d915e03c4835e0d99a27594a65559ed907f72c/cloud-add-new-cloud.png)

![pick docker](https://gist.github.com/jrrickard/114b8c35b1d5306ff3e0/raw/d7d915e03c4835e0d99a27594a65559ed907f72c/pick-docker.png)

That should give you a blank entry form to configure the Docker cloud.

![blank docker entry](https://gist.github.com/jrrickard/114b8c35b1d5306ff3e0/raw/d7d915e03c4835e0d99a27594a65559ed907f72c/blank-docker-config.png)

Enter the relevant configuration for your system:

![my system](https://gist.github.com/jrrickard/114b8c35b1d5306ff3e0/raw/6c8284d1dee92d3c8eb9a1f7565832549b8b1000/docker-jenkins-config.png)

Then create an image that matches our jenkins slave image. The defaults largely work here if you used the base image above. 

![configure an image](https://gist.github.com/jrrickard/114b8c35b1d5306ff3e0/raw/d7d915e03c4835e0d99a27594a65559ed907f72c/jenkins-docker-image.png)

Notice the label here. This is what you will is used to match Jenkins jobs to the docker setup. I also checked the "Docker Container" checkbox and the "Clean local images" checkbox. This should instruct Docker to clean up after itself.

Next, go create a job. 

![create a job](https://gist.github.com/jrrickard/114b8c35b1d5306ff3e0/raw/d52762c15357eee01b7b17fd89b10b21710c3139/deploy-job-config.png)

As mentioned, the crucial part here is to put the label entered in the Docker image config in the "Label Expression" text box and check "Restrict where this project can be run". The label here needs to match what was entered in the earlier image config.

Now when you run your job, it should execute within the Docker container. Here is my job:

![built on docker](https://gist.github.com/jrrickard/114b8c35b1d5306ff3e0/raw/4d3ee2aca02c53d4736ef3bba1bac9a7cc25578c/built-on-docker.png)


Some additional things to consider:

* No authentication on that docker connection, I might update this later with changes to that, ideally with TLS. 
* Might be able to build smaller images if you start with something other than the turn key image referenced above
* The use here isn't really a compelling story, my next docker slave will run a headless firefox browser to do automated testing of our UI. I'll blog that next


