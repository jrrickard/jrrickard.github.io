---
layout: post
title: "Setting Up Local DC/OS Lab - Part One"
date: 2016-07-23T20:22:26-06:00
---

In my new role, I'm working pretty extensively with Marathon and Mesos. Prior to moving to my home office, I setup a Marathon and Mesos cluster using my [Link text]({% post_url 2014-08-05-3d-print-all-the-things %}) lab. That lab was still in the office taking up (limited) space on someone's desk so I've still been using it, but the network connectivity between my hosue and the office meant that I had noticable delay when transfering Docker images to the docker registry running on the lab. On Friday, I decided to bring it home and set it up in my basement. 

I could keep the cluster running as is, but I've decided to reinstall it using DC/OS instead of simply installing Marathon and Mesos again. I've also decided to recreate the storage volume on the NAS, as it still contained things from my previous role that I no longer needed. This will be a totally clean rebuild of the lab then! I'll need to setup the NAS again, along with the hosts, followed by the DC/OS setup. I'll also need to run a docker registry to support what I am doing. I'll use the NAS to provide storage for the docker registry and mostly use the host storage for the Mesos agents, although I'll also do some research for persistent volumes from Mesos and will use it. It's got around 2 TB of storage if I set it up with RAID1. 

My original plan was to run [CoreOS](https://coreos.com/) on the hosts, but I don't think that really fits with my needs right now (i.e. I don't need etcd or other features of CoreOS right now), so  I'll revisit that later. 

![pxeboot]({{site.url}}/images/pxeboot.png)

I was able to setup a PXE boot server using my NAS, since Synology supports that pretty well. In the short term, I'll use [Ubuntu](http://www.ubuntu.com/) since I know that I can install that on the NUCs without much troubleshooting. Later I'll maybe move to [CentOS](https://www.centos.org/), because that will be a mirror of what we're using in production at the moment. I previously had to make some BIOS changes to get Ubuntu running on the NUCs, so I'm guessing I'll need to do something similar for CentOS.    

I currently have the hardware setup in the basement, using some old entertainment centers to hold everything. 

![lab]({{site.url}}/images/basement_lab.png)

In the next blog, I'll cover getting Ubuntu running on the hosts. After that, I'll follow up with getting DC/OS running. I currently have the bare minimum for running DC/OS, so I think I'll probably buy another NUC or two over the coming weeks and add them as Mesos agents. 

