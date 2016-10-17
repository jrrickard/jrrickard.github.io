---
layout: post
title: "Comparing Spring and Xenon"
date: 2016-10-16T20:07:21-06:00
---

In my last job, our Java development exclusively used the [Spring](https://spring.io/) ecosystem. In my current job, we're instead using [Project](https://vmware.github.io/xenon/) [Xenon](https://github.com/vmware/xenon). Both frameworks enable you to build services, but help you get there with pretty different approaches. To provide some comparison between the two, I've gone ahead and built the same application twice: one using Spring and one using Xenon. You can accomplish a LOT with both the Spring ecosystem and Xenon, but I've chosen to keep this fairly simply and compare a fairly simple application. I originally wrote the app to better learn Xenon (for the day job) and to help automate some of my volunteer responsibilities with the [Pikes Peak Road Runners](https://pprrun.org/). The app exposes a REST API for managing race and participant data for a series of winter running races the club holds. I've been acting as the registrar for the series for a few years now and the process has been fairly manual and prone to human (me) data entry errors. I wanted to make it a little easier, and learn Xenon a little more, so I wrote the app. It's extra timely because we're expecting our second child around the time of the race series, so I'll probably have much less time to volunteer this year. I've decided to offer a comparison point to Xenon using Spring since it's a pretty popular choice in the Java world and I expect many people have some familiarity with it.  

The Spring based application is written using Spring Boot, Spring MVC, Spring Data and a Mongo DB instance. I need to add everything to github still, but when I do there will be a some scripts to build the source, package as Docker and a Docker compose file that will run the app. The Xenon app, on the otherhand, is written only using the Xenon framework. Xenon makes use of Lucene as a persistence layer, so it does not need an external database. 

Stay tuned for three blog posts this week: 

1. First, I will walk through the Spring app
2. Next, I will walk through the Xenon app
3. I'll offer my thoughts comparing the two.

 
