---
layout : post
title: AWS Lambda and Alexa Skills Kit
date: 2015-09-14T07:49:23-06:00
---

A few months ago, I bought an Amazon Echo because I have a ton of music in the Amazon ecosystem. When they announced the [Alexa Skills Kit](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit), I was excited to try it out. I had already done a few things with [AWS Lambda](https://aws.amazon.com/lambda/), so I was already familiar with writing a lambda function. I decided a quick way to try it out would be to implement something with ASK that I had done previously. I recently wrote a bot for Slack using go and one of it's first functions was to do things with the Rally Web Services API. That seemed like a fun thing to do with the Echo as well. I already ask the Echo for the weather and a couple other things in the morning...what If I could ask it what bugs I have to fix? I created a github repo to store everything I describe below.
I started out by grabbing the [reference skills](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/using-the-alexa-skills-kit-samples). I already had a develoepr account, so once I had the reference skills I was able to register them on the develoepr portal and try them out. Developing a skill turns out to be **REALLY** simple. You basically come up with an interaction model, write some JavaScript, upload it to AWS, do a little configuration and it's ready to go.

Using the references as a starting point, I came up with an interaction model. The first thing I would implement with the Rally API was fetching my defects. The recommended interaction design is to ask your service for something. You might say, "Alexa, ask <Invocation Name> for <intent trigger>. I originally started out with "my current defects" for the intent trigger, but it was more natural for me to say bugs so I went with that. Getting bugs from the Echo would be:

```
Alexa, ask Rally for my current bugs
```

Rally would become the Invocation Name. We'll look at configuring that in a minute.

Now it was time to implement my lambda function. I had two choices here, I could implement a client for the Rally WS API myself, or use the [unsupported Node.js library](https://github.com/RallyTools/rally-node) that Rally developed. I went with the library to avoid reinventing the wheel. It's easy to use Node libraries with Lambda, you just need to delivery the library with the Lambda function. The easiest way to do this is to run the npm install and specify a prefix so that it installs in the same directory as your lambda code. Then you can simply zip up the entire directory and upload that to the AWS console. In my case, I had created a directory called "rally-tasks-for-echo" and a "src" directory under that to hold all the JavaScript. Installing the node library was just:

```
fermat:rally-tasks-for-echo jeremy$ npm install --prefix=src rally
```

Now, when I looked in the src directory, I had a node_modules subdirectory with the Rally library and the associated dependencies.

Next, I needed a way to configure the Lambda. I decided to create a JSON document with my read-only Rally key, my project ID and my username and stoer that in S3. This ended up being a really easy way to store the configuration. Here is an example:

```
{
        "key" : "INSERT_RALLY_KEY",
        "projectID" : "INSERT_RALLY_PROJECT_ID",
        "user" : "INSERT_USERNAME"
}
```

The actual Lambda function is pretty simple, so I won't go into it much here. Simply zip up the directory with the source code and keep the file handy. Once it was written I had to configure both the Lambda and the Skill. Once all the configuration is done, I need to come back and update the source code with the appId of the skill. The Lambda should be first.

First, login to the AWS Console. Then Click on the Lambda link. It's also important to make sure you are using the us-east region, as the other regions are not supported for ASK yet.

![aws console]({{site.url}}/images/lambda-option.png)

Then create a new Lambda function. I skipped the blueprint and just went with a blank configuration. 

![new function]({{site.url}}/images/create-lambda.png)

Now, name the function and pick Node.js as the Runtime. 

![configure-function-step-one]({{site.url}}/images/configure-function.png)

Change the code entry type to "Upload a ZIP File" and click the Upload button and locate your file. 

Once uploaded, keep the Handler as index.handler, that matches up against the handler function defined in index.js. For role, I selected the basic Lambda exeuction policy and that seemed sufficient.

The major thing I had to change was the default timeout period for the function. The Rally API can take a few seconds to return, so I upped the timeout to 20 seconds. Probably more than needed but I wanted to be safe. Something to consider is that based on Lambda pricing, increasing the timeout could have a cost impact. I don't think that I'll exceed the free invocation limits so I wasn't too worried about it here.

The final bit of configuration for the Lambda function is to add an event source to it. Edit the function by clicking on it's name, then click the "Event sources" tab and click "Add event source" once that loads. Select Alexa Skills Kit from the Event source type drop down, then click submit.

Now the Lambda is configured. Now go to the Alexa Developer Portal (https://developer.amazon.com/edw/home.html#/) and click Alexa, then Get Started under Alexa Skills Kit. Pick a name for the skill and an invocation name. As I mentioned above, I picked Rally for my invocation name. I also used that as the name. I also entered the ARN from my Lambda function here. Copy the Application Id on this page for use later. We'll need to add that to the skill lambda code.  

Next, I copy and pasted the contents of the speechAssets files into the text fields on the next screen. 

Now that the skill is set up, I should have an application id associated with it.  Take the application id and paste it into the source code from the git repo above, replacing the template value that is there. Upload the code again and you should be able to test it with your Echo.

Here is mine: 

<iframe width="420" height="315" src="https://www.youtube.com/embed/KfkRT0aDw5o" frameborder="0" allowfullscreen></iframe>


