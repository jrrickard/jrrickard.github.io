---
layout: post
title: "Running lattice.cf on VMware AppCatalyst"
date: 2015-06-23T19:12:53-06:00
---

I'll preface this by saying I don't think any of this is supported and AppCatalyst is a Tech Preview. But it's a great developer tool!

First, install [AppCatalyst](https://communities.vmware.com/community/vmtn/devops/vmware-appcatalyst)

Next, install the [AppCatalyst Vagrant Provider](https://github.com/vmware/vagrant-vmware-appcatalyst)

The Vagrant Provider needs the AppCatalyst REST API, so once installed, you need to start the AppCatalyst daemon: appcatalyst-daemon

![start daemon](https://gist.github.com/jrrickard/a2724c9ede44de0186cc/raw/f63b7706bef41312eb9ad7a947e2f72e4dde94c1/start_deamon.png)

Finally, follow the [Lattice.cf getting started guide](http://lattice.cf/docs/getting-started/), including installing ltc. 

Once you get everything installed, the only real issue I had was networking. 

The default subnet for AppCatalyst is 192.168.136.0, per Library/Preferences/VMware AppCatalyst/networking:

~~~~
answer VNET_8_HOSTONLY_NETMASK 255.255.255.0
answer VNET_8_HOSTONLY_SUBNET 192.168.136.0
~~~~

However, the Vagrantfile looks for an environment variable that contains the Lattice IP or it defaults to 192.168.11.11. The IP seems to increment by 1 for each new virtual machine in AppCatalyst, so my next IP would have been 192.168.136.133.

So finally, I launched the vagrant box with: 

~~~~
LATTICE_SYSTEM_IP=192.168.136.133 vagrant up --provider=vmware_appcatalyst
~~~~

![vagrant up](https://gist.github.com/jrrickard/a2724c9ede44de0186cc/raw/e831083aef54bcfcb9a38de1878a158cd8ff11ae/vagrant-up.png)

Once that was up, run the ltc target:

~~~~
jrickard-mbpro:lattice jrickard$ ltc target 192.168.136.133.xip.io
Api Location Set
~~~~

Finally, I launched the sample lattice-app. 

~~~~
jrickard-mbpro:lattice jrickard$ ltc create lattice-app cloudfoundry/lattice-app
No port specified, image metadata did not contain exposed ports. Defaulting to 8080.
No working directory specified, using working directory from the image metadata...
Monitoring the app on port 8080...
No start command specified, using start command from the image metadata...
Start command is:
/lattice-app
Creating App: lattice-app...................
06/22 19:54:01.92 [APP|0] Successfully created container
06/22 19:54:02.22 [APP|0] {"timestamp":"1435024442.223613262","source":"lattice-app","message":"lattice-app.lattice-app.starting","log_leve
l":1,"data":{"port":"8080"}}
06/22 19:54:02.22 [APP|0] {"timestamp":"1435024442.223858833","source":"lattice-app","message":"lattice-app.lattice-app.up","log_level":1,"data":{"port":"8080"}}
06/22 19:54:02.75 [HEALTH|0] healthcheck passed
06/22 19:54:02.75 [HEALTH|0] Exit status 0
lattice-app is now running.
App is reachable at:
http://lattice-app.192.168.136.133.xip.io
~~~~

![targeted](https://gist.github.com/jrrickard/a2724c9ede44de0186cc/raw/77b99e563818b87bc279a4b32052182de100ebff/ltc-target_and_ltc-create.png)

Now, let's fire up a browser and hit http://lattice-app.192.168.136.133.xip.io

![we have app](https://gist.github.com/jrrickard/a2724c9ede44de0186cc/raw/c6289cc2872f9ba7014bb76cb54d3ce0ef5348be/lattice_app.png)


