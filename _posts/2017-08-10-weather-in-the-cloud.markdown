---
layout: post
title: "Weather in the Cloud"
date: 2017-08-10T08:40:33-06:00
---

This week I did a talk about [Xenon](https://github.com/vmware/xenon) at our local Open Source meetup. I finished up most of my Spring vs Java stuff to use there, so I'll hopefully get a blog written about it soon and tie that up!

I've decided I need a more interesting project to work on at home. Between work and having a new baby, I haven't really touched any of my Arduino / Raspberry Pi stuff in quite a while. I miss tinkering with that stuff and I just discovered I had ordered a bunch of parts to make a weather station and then never did anything with it. I've also decided that I want to experiment with [Azure](https://azure.microsoft.com/en-us/), since I've done a lot with AWS and GCP in my day job and nothing with Azure. So I think what I'll do is assemble the weather station and have it publish data to a little Raspberry PI that I'm using as an IOT Gateway sort of thing for some other projects that have been just running without any attention for a long time. Then I'll have the rPi send data to Azure. I think this will let me get some experience with Azure from a compute perspective (maybe using their cool new container service), storage perspective, and maybe Azure functions to do some "alerting" with the data. Should be a fun little project. 


