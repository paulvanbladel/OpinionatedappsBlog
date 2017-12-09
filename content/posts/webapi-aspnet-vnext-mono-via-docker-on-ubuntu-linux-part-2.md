+++
date = "2015-02-25T08:01:08+00:00"
draft = false
title = "WebApi AspNet.Vnext Mono with Entity Framework via docker on Ubuntu Linux (part 2/n)"
slug = "webapi-aspnet-vnext-mono-via-docker-on-ubuntu-linux-part-2"

+++

##Introduction
In this article, we'll start from a fresh Ubuntu server and get a .Net web api up and running.
All this can be covered in this very post. We'll use the source code from the [previous post](http://blog.opinionatedapps.com/webapi-aspnet-vnext-mono-via-docker-on-ubuntu-linux-part-1/) stored on github [over here](https://github.com/paulvanbladel/AspNet5-Web-Api-Sample/tree/blog-sample).

##Step 1: Provision an ubuntu server
This is very easy and covered everywhere on the net. I'll use a virtual machine on my local windows 8 laptop using Hyper-V, but you can create an instance on azure, DigitalOcean, you name it.
Make sure you can ssh into your server. When using a cloud based vm, ssh will be enabled by default but not when using hyper-V. 

```language-bash 
sudo apt-get install openssh-server
```
Obviously, you can simply use the hyper-v client to login to your vm, but I prefer to use another SSH client namely [BitVise](https://www.bitvise.com/ssh-client). The reason is that the support for copy pasting to the vm is not good on Hyper-V.
## Step 2: Install Docker on the server
Don't use the standard procedure here via apt-get install docker.io, because that gives you an older version of docker but proceed as follows:
```language-bash 
curl -sSL https://get.docker.com/ubuntu/ | sudo sh
```
Let's check now we have the latest version (currently 1.5):
```language-bash 
sudo docker --version
```
```
Docker version 1.5.0, build a8a31ef
```
## Step 3: Create the dockercontainer for your web api app
```language-bash 
sudo docker run -it -p 5000:5000 microsoft/aspnet /bin/bash
```
This might take a minute or 2. So this process will create kind of mini vm inside your server where we will eventually run our aspnet web api app. The command indicates that we will run this container interactive (we have /bin/bash available) and we will expose port 5000 to the external world. More details on [docker.com](https://docs.docker.com/)
Note that our command prompt changed now, because we are not in the container:

```language-bash 
root@ddc8d4aeac72:/#
```
## Step 4: fine tune the container
This step should be automated in a later post, but in order to get a good understanding of what's happening, we'll do it manually. Typically a Dockerfile is used for automating this step.
Since we are using the microsoft/aspnet Docker image, everything is already installed for running .Net in this container. Though, we still need a few additional things. First of all, we need Git and we'll install also a text editor called nano:
```language-bash 
 apt-get install git nano -y
```
Notice we no longer need to sudo the command, since we run as root in the container. the -y flags answers all questions from apt-get with yes :)
Ok, now we are ready to download our .net project from github. Let's first make a directory for your github repositories.
```language-bash 
mkdir repos
cd repos
git clone  https://github.com/paulvanbladel/OwinMonoDocker.git
cd OwinMonoDocker
```
## Step 5: some Nuget tweaks
In fact we are ready now to build our project (from the OwinMonoDocker folder) via:
```language-bash 
xbuild OwinMonoDocker.sln
```
but this still will fail because we need to restore the nuget packages used in the visual studio solution. So let's try that:
```language-bash 
nuget restore OwinMonoDocker.sln
```
That will fail as well, because Nuget isn't configured correctly, it needs to know which sources to use. So, we need to update the Nuget configuration file located in /root/.config/NuGet. Ok, let's use our text editor nano:
```language-bash 
nano /root/.config/NuGet/NuGet.Config
```
and copy paste following nuget source locations in that file:

```language-markup
<?xml version="1.0" encoding="utf-8"?> <configuration>
<packageSources>
<add key="nuget" value="https://www.nuget.org/api/v2/"/>
<add key="aspnetwebstacknightlyrelease"
value="http://www.myget.org/f/aspnetwebstacknightlyrelease/"/>
<add key="aspnetwebstacknightly"
value="http://www.myget.org/f/aspnetwebstacknightly/" />
<add key="azureadwebstacknightly"
value="http://www.myget.org/F/azureadwebstacknightly/"/>
</packageSources>
</configuration>
```
Go back to the OwinMonoDocker folder and check now if nuget has well digested that new config file:
```language-bash 
nuget sources
```
It should give back the nuget sources configured above. This step is definitely something which can be automated via the Dockerfile mechanism.
## Step 6: run nuget restore and build the project
```language-bash 
nuget restore OwinMonoDocker.sln
```
Check carefully all packages are installed, otherwise the project won't run.
Xbuild now the sources
```language-bash 
xbuild OwinMonoDocker.sln
```
If the process ends with:
```language-bash 
Done building project "/repos/OwinMonoDocker/OwinMonoDocker.sln".
Build succeeded.
         0 Warning(s)
         0 Error(s)
```
everything should work now 
## Step 7: run the executable
```language-bash 
mono OwinMonoDocker/bin/Debug/OwinMonoDocker.exe
```
```
Starting web Server...
Server running at http://*:5000 - press Enter to quit.
I'm running on Unix 3.16.0.30 directly from assembly OwinMonoDocker, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null
```
## Step 8: test the api
In my case, using an hyper-v virtual machine, I can browse to the IP address of the vm and test directly the web api:
```
http://<ip address of your vm>:5000/api/Customers/
```
That should give back a whole bunch of customers.

## Next steps
I cheated a bit with the database connection. In the current implementation, you'll directly connect to my personal MySql database (sure, via entity framework). So, if you are reading and testing this and things wouldn't work it might be because I did shut down the database :) In a later post we'll setup the database server via a dedicated Docker container and let these containers play together.

## Conclusion
I think Docker is quite impressive. Without going too much in details, do you know that the above docker image with everything installed including the web api project only takes 87 mega bytes? I'm impressed, this changes the way we can think about an application and service eco system. For me really the first time that something which is in essence more IT infrastructure related has such a large impact on the way I think about software architecture. More on that later.

 

