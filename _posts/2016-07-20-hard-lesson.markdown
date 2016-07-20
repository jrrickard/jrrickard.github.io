---
layout: post
title: "Painful Lesson"
date: 2016-07-20T10:42:45-06:00
---

We are using [Mesos](http://mesos.apache.org/) and [Marathon](https://mesosphere.github.io/marathon/) to run our Dockerized system. In the matter of two days, I learned a painful lesson. Namely, the following is a terrible, terrible practice:

```json
  "container": {
    "docker": {
      "image": "repo/image-name:latest",
      "forcePullImage": true,
      "network": "BRIDGE",
      "portMappings": [
        {
          "containerPort": 8000,
          "hostPort": 0
        }
      ]
    }
  }
```

Using forcePullImage : true and the latest tag on the image goes really badly when you implement CD and push images with breaking changes to your repository. In our current deployment scenario, we run a cluster of three containers. The implication with forcePullImage : true is that anytime the container is restarted, it will pull the latest of whatever image is specified and start with that. That is GREAT for testing out new changes when you manually initiate a restart across the cluster. That is TERRIBLE when one of the containers dies and restarts with the new image, leading to an inconsistent cluster. This of course happened overnight in our staging environment!

We're now pushing images that make it through our image promotion process with specific tags and we are not using the forcePullImage setting. This poses an issue: what if we DO want to have it pull the image? Really, the solution here is another tag. If there is a problem with the image, it should be replaced with a new verison. 

If you DO need to force an image pull, you can change the Marathon configuration to specify that, causing a restart (after the pull). You'd likely want to then change it again to change it back, but this would also result in a restart. Another option might be to run a non-container task and execute 'docker pull' on each of the Agents (slaves in old Mesos terminology). That might look something like:

```
{
  "id": "test-pull",
  "instances": 3,
  "mem": 128.0,
  "cpus": 1,
  "cmd": "sudo docker pull %%IMAGE_NAME%%",
  "constraints": [
    [
      "hostname",
      "UNIQUE"
    ]
  ]
}

```

In this case, you'd want to replace the instances attribute with the number of agents in your cluster. The hostname : UNIQUE constraint should force this to run on each of the agents.


This probably isn't the greatest use of Marathon, as it will run this task over and over again. Once it is run, you probably want to delete the app from Marathon to prevent this. A better alternative might be to use [Chronos](https://mesos.github.io/chronos/) or [Eremetic](https://github.com/klarna/eremetic) 
