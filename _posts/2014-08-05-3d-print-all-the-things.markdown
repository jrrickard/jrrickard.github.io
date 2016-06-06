---
layout: post
title: "3D Print All The Things"
date: 2014-08-05 18:45:28 -0600
comments: true
categories: 3D-Printing,NUC,Lab 
---

When Josh and I got accepted to our [Internal R&D Conference](http://cto.vmware.com/radio-a-different-kind-of-high-performance/), I bought two [Intel NUC](http://www.intel.com/content/www/us/en/nuc/nuc-kit-d54250wyk.html) boxes to run ESX and a few VMs on. After RADIO was over, I kept them on my desk at work and they turned out to be super useful for the project we're working on right now. So useful that we ended up supplementing them with a small NAS and two more NUC boxes. Now we have a little mini lab sitting on my desk with 64 GB of RAM, 4 4th generation Intel® Core™ i5-4250U processors, a terabyte of Solid State Storage and 4 Terabytes of spinning disks. Pretty impressive for the size and energy footprint. The processors are obviously the short pole in the tent, but it's been working very well for a little test enviroment running the vCenter Appliance, the new Log Insight (which is totally awesome), a few instances of our project and a couple of random Ubuntu Virtual machines. 

The only problem is that now I had four NUCs and a NAS just sitting on my desk. The NUC boxes have little rubber feet and sit on each other well but we thought we could do better. And we had a 3D printer in the office.

So we designed a rack for the NUCs!

![nuc rack]({{site.url}}/images/nuc_rack.jpg)

An earlier incarnation of the rack next to the Synology NAS. 

![desk lab]({{site.url}}/images/desklab.jpeg)
<!--more-->

The rack is six pieces: four identical legs and two identical base plates (top and bottom). Once assembled, you get a nice little rack that holds 4 NUC boxes and also provides a little more airflow. 

I placed the STL files for both parts in a [github repository](https://github.com/jrrickard/NUC-RACK). The cool part of this is that last year, github added the ability to [view STL files](https://github.com/blog/1465-stl-file-viewing) using a combination of Three.js and WebGL, so you can preview the pieces directly from the github repo.

Here are a couple of screen shots from my browser:

![stl file for rack legs]({{site.url}}/images/github-stl-rack-legs.png)
![stl file for the baseplate]({{site.url}}/images/github-baseplate-stl.png)

If you'd like to make a version of this, grab the STL files and adjust the leg pieces for the number of NUCs you have. This supports four and was about the maximum we could print on our printer. Once you print the pieces, the legs pop into the slots on the base plates. Each leg has small platforms that the rubber feet on the NUC will rest on. You could even take these pieces and make them a little bigger with your favorite CAD program (we used [Tinkercad](http://www.tinkercad.com) to make these) and use them to hold something with a similar form factor, like a Mac Mini. 

For comparison, here is a piece on a 7.5-inch by 4.75-inch [Field Notes](http://fieldnotesbrand.com/) Arts & Science notebook. 

![a printed rack piece]({{site.url}}/images/rack_piece.jpg)
