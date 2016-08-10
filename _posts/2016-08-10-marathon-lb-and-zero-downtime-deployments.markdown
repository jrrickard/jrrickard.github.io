---
layout: post
title: "Marathon-LB and Zero Downtime Deployments"
date: 2016-08-10T15:51:48-06:00
---

We're currently using Marathon-lb to act as a sort of service discovery for our application and the services we are deploying. With our application, we need to deploy a new cluster for our application and initiate a data migration when we need to update. This poses a problem with our current use of Google's Load Balancer and Marathon-lb, namely the use of service ports. After trying a few things, we settled on creating another load balancer pointing at a new service port and swapping our DNS records. That's somewhat time consuming though and extra for us to automate for upgrades. After reading the [Zero Downtime Deployments](https://github.com/mesosphere/marathon-lb/blob/master/README.md#zero-downtime-deployments) section of the Marathon-lb documentation, it sounded like a decent fit for what we needed. Not a 100% fit given our deploy->migrate upgrade sequence, which can take 15+ minutes to complete, but close. I decided to try to get the Marathon-lb portion of this working using the HAPROXY_ labels described in the documentation.

This failed with lots of Python errors, and I believe it is due to the older version of Marathon-lb we are running. We haven't fully migrated our production environments to DC/OS yet, and we're running a version of Marathon-lb that is quite old in the scope of Marathon's life. I decided to try this out with my DC/OS lab and spun up the latest Marathon-lb instance. I decided to run this outside of Marathon because I was experimenting with a number of config options and it was easier to just run via the command line....and I only have one agent currently so that's not a big deal. I've settled on the config I want, so I can migrate it back into Marathon now. 

Following the steps described MOSTLY worked. I simply changed our JSON definition to include:

```
HAPROXY_DEPLOYMENT_GROUP=csp
HAPROXY_DEPLOYMENT_ALT_PORT=10101
```

Running that, I saw it pick up both applications after I deployed the new containers:

```
marathon_lb: fetching apps
marathon_lb: GET http://master.mesos:8080/v2/apps?embed=apps.tasks
marathon_lb: got apps ['/csp-host', '/csp-host-2']
```

However, it immediately started spewing Python errors:


```
marathon_lb: Unexpected error!
Traceback (most recent call last):
  File "/marathon-lb/marathon_lb.py", line 1357, in do_reset
    self.__apps = get_apps(self.__marathon)
  File "/marathon-lb/marathon_lb.py", line 1178, in get_apps
    int(new['labels']['HAPROXY_DEPLOYMENT_TARGET_INSTANCES'])
KeyError: 'HAPROXY_DEPLOYMENT_TARGET_INSTANCES'
```

The documentation suggested that wasn't something you should NEED to set, but it showed up nonetheless. I'm guessing that because I was not using the Zero Downtime Deployment scripts, I needed to provide this. This was pretty easy to fix, I just needed to add that label to the JSON config.

```
HAPROXY_DEPLOYMENT_GROUP=csp
HAPROXY_DEPLOYMENT_TARGET_INSTANCES=3
HAPROXY_DEPLOYMENT_ALT_PORT=10101
```

After adding that and waiting for the app to restart, I was able to access both of my application clusters via the same service port.

Since we can't really use the ZDD script, I made my own Python script that handles our use case....namely:

1. Fetch a task from the old cluster
2. Get the IP and Port combo
3. Grab a task from the new cluster
4. Get the IP and Port combo
5. Build an upgrade request to the new task using the old task as the source node
6. Wait for the upgrade request to finish
7. Modify the new cluster to add the tags above.
8. Trigger a tag of the image in Bintray to represent our currently deployed state

The last step was really the important part to keep the new cluster from receiving traffic before the upgrade is complete. Once the upgrade is complete and we've verified the new instance, the old one can be removed from Marathon. 
 
