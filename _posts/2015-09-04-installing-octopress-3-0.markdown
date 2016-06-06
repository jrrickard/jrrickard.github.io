---
layout: post
title: "Using Octopress 3.0 with Github Pages"
date: 2015-09-04T20:14:30-06:00
---

This blog has been dormant for a while. I was using a fairly old version of octopress and at some point ended up corrupting my repo. Then I got way to busy with the [vRealize Air Compliance](http://vrealizeair.vmware.com/compliance) to fix it. Now that we've released, I thought it was time to get this thing running again. 

[Octopress 3.0](https://github.com/octopress/octopress) is available on github, so I thought I'd try that out. Rather than try and upgrade my existing instance, I went ahead and killed my github pages repo and start from scratch there. I also had a new Macbook, so I might as well try and set everything up from scratch.

#Step one, create a new repo

To use github pages, you create a repo of the form username.github.io. In my case, that was jrrickard.github.io. Next, I followed the [Usng Jekyll with Pages](https://help.github.com/articles/using-jekyll-with-pages/) page to get started up to the "Running Jekyll" section of the page.

#Step two, get Octopress

I modified the Gemfile created above and added:

~~~~
gem 'octopress', '~> 3.0.0'
~~~~

Once I did that, I did a bundle update && bundle install to make sure I had the new gem. Now I had the octopress cli. This has been *much* easier to use than the previous version of octopress. Now I had everything installed, I needed to init the new blog. You can do that with Jekyll, or you can use the octopress cli and it will also copy over all the octopress scaffolding. I went with the octopress:

~~~~
# octopress new jrrickard.github.io
~~~~ 

That created a new directory called jrrickard.github.io with the scaffolding. Next I changed into that diretory and did a jekyll build

~~~~
# jekyll build
~~~~

This went through the directory and created the _site directory with all of the content of the blog. Now I could serve it up:

~~~~
# jekyll serve
~~~~

This started up a server listening on port 4000 so I could view the content. 

The last step of getting it setup was doing the deploy init. Again, very easy with the octopress cli:

~~~~
# octopress deploy init git git@github.com:jrrickard/jrrickard.github.io.git
~~~~

Once that was done, I added the _deploy.yml file to my .gitignore. Once I was done, the .gitignore looked like:

~~~~
_site
.sass-cache
_deploy.yml
~~~~

Finally, running deploy with the octopress cli pushed to my git repo:

~~~~
# octopress deploy
~~~~

Now, the new blog was available at my Github pages url. Now I just needed content!

#Step three, move all the old content.

This part was simple. I still had all the markdown and image files from my original blog, so I moved them over to this new directory structure and put the existing markdown files in the _posts directory. I created an images folder and copied all the images over into that one. Once I ran the jekyll build to regenerate everything, I noticed I didn't have the old octopress img plugin installed. I went ahead and replaced those with normal markdown and the Jekyll site.url template (make sure you update your _config.yaml to have that correctly defined). 
 
#Step four, new posts!

While my blog was messed up, I had done a few gist based blogs. I wanted to move them over here, so I used the cli to create some new posts:

~~~~
# octopress new blog "Running lattice.cf on VMware AppCatalyst"
# octopress new blog "Creating a Docker Jenkins Slave Running on a VMware Photon VM"
~~~~

This created two new files in the _posts directory with that title and today's date. It sticks the correct header/front matter in a blank file. So I modified the filename to have the correct dates and updated the date meta data in the file. Then I simply copied the markdown from the Gists and placed them in the appropriate file. The only major change I had to do was to change the Github markdown preformat token (three `) to normal markdown (4 ~). Once I had done that, getting everything deployed again was simply:

~~~~
# jekyll build
# octopress deploy
~~~~

Then voila, the blog is updated. Simple.


