+++
date = "2015-01-15T12:13:48+00:00"
draft = false
title = "Adopting a lean web app server infrastructure based on nginx (part 1/3)"
slug = "adopting-a-lean-web-app-server-infrastructure-part-1"

+++

##Introduction
Modern web applications build with **isomorphic JavaScript** frameworks (JavaScript both on client and server) can be easily deployed to web servers other than Microsoft's IIS. In fact, there [are ways to run NodeJs](https://github.com/tjanczuk/iisnode) on IIS but, without going here too deep into technical details, that really feels for me like a very strange marriage. 
But wait, even when you want to stick to a **.Net based application**, [you can use nowadays Linux instead of a Windows server](http://www.hanselman.com/blog/AnnouncingNET2015NETasOpenSourceNETonMacandLinuxandVisualStudioCommunity.aspx).
In short, reasons enough to explore this (new) route. 
##Our (playground) infrastructure
Before starting to talk about nginx (so the counterpart of IIS on Linux, well at least my favorite web application server on Linux, there are other options), let's first see how you can set up at a very low cost a Linux server that is available over the internet.

##Option 1 : a Raspberry pi
No, this is not a joke. I'm currently running a raspberry pi at home with Linux installed (pretty much the only option) and it has, among others, nodeJs and  MongoDb installed.
The [raspberry pi](http://www.raspberrypi.org/) is simply connected to my home router and luckily my internet provider allows me to use port 80, so it's publicly available. Obviously, I did set up the DNS accordingly.
The following pictures show my pi in my office. I should definitely buy a decent case for that little baby.

![My raspberry pi at home](http://blog.opinionatedapps.com/content/images/2015/01/RaspberryPi.jpg)

I installed, as kind of test a copy of my previous pragmaswitch blog (which I imported in Ghost).
Currently (January 2015) it's available [here](http://home.opinionatedapps.com/), but I might use it later on for other purposes. You might notice it runs pretty fast !
Although you could, I'm not pretending you should install production applications on a raspberry pi, but it is definitely a very good choice as staging environment. Since it has a low cost processor, you will notice quite easily performance problems in your application. Even for a blog (in production mode) it would be a great choice if you want a real very low cost solution. You only need to work out a decent backup strategy. It's really low cost because you have not the variable cost of a hosting provider and when it comes to energy consumption: a raspberry pi is connected to a cell phone charger and runs on 5 Volt, so Greenpeace will not protest !
##Option 2: azure
Microsoft understood quite well the power of Linux and they offer nowadays great support for e.g. Ubuntu servers.
Let's quickly walk through the steps to setup an Ubuntu server. It's very straightforward.
######Create a new virtual machine from the azure management portal
![Azure](http://blog.opinionatedapps.com/content/images/2015/01/Azure1.png)
######Select the Ubuntu server template
![Azure](http://blog.opinionatedapps.com/content/images/2015/01/Azure2.png)
######Provide a name and user
You can work with a user name/password combination or optionally provide directly an SSH key.
It's always possible to add the RSA public key afterwards and disable login via username/password on the Linux box. In case you are not familiar with generating RSA key pairs, it's best to go first for the username/password approach. But note that for serious work, you should definitely use RSA !
![Azure](http://blog.opinionatedapps.com/content/images/2015/01/Azure3.png)
######Select a region
![Azure](http://blog.opinionatedapps.com/content/images/2015/01/Azure4.png)
######Add an endpoint
![Azure](http://blog.opinionatedapps.com/content/images/2015/01/Azure5.png)
######Select http as endpoint
![Azure](http://blog.opinionatedapps.com/content/images/2015/01/Azure6.png)
In case you plan to use SSL on your web app server, add also the 443 port for SSL.
######Log into your server via ssh
The best choice, when connecting from a windows machine, is here to use [putty](http://www.putty.org/). In case you connect from a mac or a Linux machine, SSH is available directly from the command propmt.
![Azure](http://blog.opinionatedapps.com/content/images/2015/01/Azure7.png)


##Option 3: digital ocean
My favorite ! [Digital Ocean](https://cloud.digitalocean.com) is definitely a more democratic option. Their plans starts from 5$ a month for full root access to a Linux box making use of a solid state drive !
The steps to create a 'droplet' (that's their name for a virtual machine) is so simple that I won't go further into detail here.

##What's next?
In the next episode, we'll install our web server on the Linux box and install something on it. But we'll also explain why I selected Nginx. Which option you selected above is irrelevant.
Stay tuned !









