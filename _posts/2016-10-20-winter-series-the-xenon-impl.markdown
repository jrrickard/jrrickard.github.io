---
layout: post
title: "Winter Series - the Xenon implementation. Part 1"
date: 2016-10-20T19:28:41-06:00
---

This week ended up being a little crazy and I haven't had enough time to really devote to writing up the Xenon vs Spring comparison, so I'm going to break this up into more managable pieces. 

I won't get into [Xenon](https://github.com/vmware/xenon) fundamentals too much in this post, but in general you build capabilities using  cooperating REST services. Xenon lets you build REST services, but also provides a mechanism for persistence and replication of data as well as built in query capabilities. 

In the Xenon world, services fall into two main types of services: stateful and stateless. At the heart of stateful services is something called a Plain Old Data Object, or a PODO. Under the covers, Xenon uses Lucene as a persistence mechanism and provides services on top of Lucene to handle replication to enable building distributed stateful services across many Xenon nodes. Within a stateful service, you implement logic to handle the persistence/management of a PODO. Stateful services can also be used to model Tasks, via [Task Services](https://github.com/vmware/xenon/wiki/Task-Service-Tutorial). Stateless services, on the other hand, do not represent some sort of state like a Stateful service. 

Some important things to keep in mind with Xenon is that it isn't really intended to facilitate building things using normal object oriented design. It discourages inheritance and focuses on code-reuse through composition of stateless helper services or stateless utility classes. It also very highly discourages blocking operations. Working with within these constraints took some adjustment for me. When I first started implementing the Winter Series app, I made the mistake of emebedding logic into stateful services (base classes) and tried to share this into my services through inheritance. This didn't work that well in practice and also caused some issues with testing the code. Writing testable code when I violated the Xenon principles actually became pretty difficult and more or less forced me into writing code that I think fit a little better with the model. As an example, I had wrote some Task services that did some blocking calls to a client library that called an external service. Writing unit tests for the Task service was actually quite difficult because I could not Mock the service easily. This led me to refactor into a stateless utility class that I interacted with via a stateless helper service.  That was much easier to provide a mock implementation of, which I think led to more testable code. 

Let's take a look at the building blocks of my Winter Series app, starting with a Participant. I need to be able to do CRUD operations of race participants, so we need a PODO and a Service object to deal with it. 

First, the PODO:

```java
package io.github.jrrickard.ws.state;

import com.vmware.xenon.common.ServiceDocument;
import com.vmware.xenon.common.ServiceDocumentDescription;

import java.util.Date;

public class Participant extends ServiceDocument {

    enum Gender {
        MALE,
        FEMALE
    }

    @UsageOption(option = ServiceDocumentDescription.PropertyUsageOption.AUTO_MERGE_IF_NOT_NULL)
    public String firstName;

    @UsageOption(option = ServiceDocumentDescription.PropertyUsageOption.AUTO_MERGE_IF_NOT_NULL)
    public String lastName;

    @UsageOption(option = ServiceDocumentDescription.PropertyUsageOption.AUTO_MERGE_IF_NOT_NULL)
    public Date dateOfBirth;

    @UsageOption(option = ServiceDocumentDescription.PropertyUsageOption.AUTO_MERGE_IF_NOT_NULL)
    public Gender gender;

    @UsageOption(option = ServiceDocumentDescription.PropertyUsageOption.AUTO_MERGE_IF_NOT_NULL)
    public String city;

    @UsageOption(option = ServiceDocumentDescription.PropertyUsageOption.AUTO_MERGE_IF_NOT_NULL)
    public String state;
}

``` 

As you can see, this is pretty simple. The AUTO_MERGE_IF_NOT_NULL enables me to write a little bit less logic in the service.

Now for the service:

```java
package io.github.jrrickard.ws.service;

import com.vmware.xenon.common.StatefulService;
import io.github.jrrickard.ws.state.Participant;

public class ParticipantService extends StatefulService {

    public static final String FACTORY_LINK = "/ws/participants";

    public ParticipantService() {
        super(Participant.class);
        super.toggleOption(ServiceOption.PERSISTENCE, true);
        super.toggleOption(ServiceOption.REPLICATION, true);
        super.toggleOption(ServiceOption.OWNER_SELECTION, true);
    }
   
}

```

This simple service gives me the Create (via a POST operation to /ws/participants), Retrieve (via GET operation to /ws/participant/id), Update (via a PUT operation) and Delete (via a DELETE operation), inherited from the StatefulService class. The PUT operation will simpy replace the state with whatever is passed as the body of the PUT. If I want to change any of this behavior, I can just override any of the methods from the StatefulService class. 

With the two above, I just need to build a Xenon ServiceHost. This is another pretty simple class:

```java 
package io.github.jrrickard.ws;

import com.vmware.xenon.common.ServiceHost;
import com.vmware.xenon.services.common.RootNamespaceService;
import io.github.jrrickard.ws.service.ParticipantService;

public class WinterSeries extends ServiceHost {

    @Override
    public ServiceHost start() throws Throwable {
        super.start();
        startDefaultCoreServicesSynchronously();
        super.startService(new RootNamespaceService());
        super.startFactory(new ParticipantService());
        return this;
    }

    public static void main(final String[] args) throws Throwable {
        final WinterSeries ws = new WinterSeries();
        ws.initialize(args);
        ws.start();
    }
}

```

Running this will start up the Participant factory service (POSTing to this will create new service instances -- one for each participant). I started this up by running the service host via the shaded jar produced by the Gradle build like so:

```bash
$ java -jar build/libs/ws-xenon-1.0-snapshot-standalone.jar --sandbox=/tmp/ws
[0][I][1477019153298][8000][startImpl][ServiceHost/0f68f721 listening on http://127.0.0.1:8000]
[1][I][1477019155321][8000/core/node-groups/default][mergeRemoteAndLocalMembership][State updated, merge with 61c60cff-fa9e-4362-a6b5-3e9863f39693, self 61c60cff-fa9e-4362-a6b5-3e9863f39693, 1477019155320001]
[2][I][1477019158320][8000/core/node-selectors/default][checkAndScheduleSynchronization][Scheduling synchronization (1 nodes)]
[3][I][1477019158320][8000/core/node-selectors/default-3x][checkAndScheduleSynchronization][Scheduling synchronization (1 nodes)]
[4][I][1477019158320][8000][scheduleNodeGroupChangeMaintenance][/core/node-selectors/default 1477019158320001]
[5][I][1477019158321][8000][scheduleNodeGroupChangeMaintenance][/core/node-selectors/default-3x 1477019158321012]
```

As you can see from the output, this started up on port 8000. You could run it without the sandbox argument, but I needed that because I've run another service on this laptop and it clashed with the state in my service (more on that later). 

I can then use a browser to examine the service :

![Query browser]({{site.url}}/images/xenon/query-in-browser.png) 

Next, I'll use Postman to create a record

![Postman create]({{site.url}}/images/xenon/xenon-post.png)

Now if I query again (using Postman again..)

![Postman GET]({{site.url}}/images/xenon/get-participants.png)

What happened here, is that first we made a POST to the ParticipantService factory. The body of this was then turned into a ParticipantService instance, with the state represented by our Participant PODO. Xenon took care of creating this for me through the StatefulService base code. When I then issued the GET request to the /ws/participants endpoint, it did a query to find all ParticipantService instances. In this case, there is only one.

I can then issue a GET request to get that specific service.

![Postman GET Service instance]({{site.url}}/images/xenon/specific-record.png)

This time, the request was handled by the default handleGet() method in StatefulService. Nice an easy to build a really simple CRUD service.
I could easily run this as a multi-node cluster now and the use of the options in the ParticipantService constructor would tell Xenon to take care of replication and owner selection for me. You can also see the default pieces of data from the ServiceDocument, like the version. Xenon will also handle that for you, allowing you to query for different versions of your service instance. If I issue changes to the service instance, that version counter will change automatically. 

There isn't a LOT of boiler plate code here and it was fairly simple to make this single object, but it is a simple case. In the next post, I'll show a more complicated example involving links between PODOs and collections, as well as building out the app a little more with an architeture that fits a little more with what we've done on my project at work. I'll also make some changes to better handle things like that default pattern for the date of birth and add in validation.  
