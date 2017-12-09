+++
date = "2015-09-10T14:46:30+00:00"
draft = false
title = "Running a asp.net 5 web api side by side on windows and linux"
slug = "running-a-asp-net-5-web-api-side-by-side-on-windows-and-linux"

+++

## Introduction
In a [previous series of two articles](http://blog.opinionatedapps.com/webapi-aspnet-vnext-mono-via-docker-on-ubuntu-linux-part-1/) I described a sample web api making use of Owin and deployed everything on linux inside a docker instance.
Microsoft is doing a great effort in bringing .Net to the next level by allowing to run .Net on the 3 major OS platforms.
What is new in this post compared to the original articles mentioned above is the fact that today we can make make use of 3 new tools: DNVM, DNX and DNU to give cross platform deployment a better experience.

## The DNVM, DNX and DNU
In case the .Net Version Manager, the .Net Execution Environment and the .Net Development Utilities are new to use, read following excellent[ codeproject article.](http://www.codeproject.com/Articles/1005145/DNVM-DNX-and-DNU-Understanding-the-ASP-NET-Runtime)

The good news is that all these new tools run both on windows, linux and IOS.

## Get the sample project from Github
You can find [here](https://github.com/paulvanbladel/AspNet5-Web-Api-Sample) the sources and enjoy the new lean solution structure of asp.net version 5. Yeah, ultimately things are getting better in the Microsoft development world...
The details of the new AspNet 5 can be found elsewhere on the net

## The revival of the command line
Inside  project.json wen can define now 'commands' that we need during development or deployment:
```language-bash
  "commands": {
    "web": "Microsoft.AspNet.Hosting --config hosting.ini",
    "ef": "EntityFramework.Commands",
    "Kestrel": "Microsoft.AspNet.Hosting --server Microsoft.AspNet.Server.Kestrel --server.urls http://localhost:5000"
  }
```

for example the ef (Entity Framework) tool will allow us to do the well known code-first database migrations:
```language-bash
dnx ef migrations add MyFirstMigration
```
## Bootstrapping on Windows
Obviously, when using visual studio as development environment, the simplest way to start your project is using IIS Express:

![](/content/images/2015/09/IisExpress.png)

When running outside visual studio, you can start now as well via:
```language-bash
dnx web
```
which will kick-off your project in self-hosted mode.

## Bootstrapping on Linux
On windows we have DNVM, DNX and DNU available since we installed visual studio. Obviously, you can install them separately (that's what you would typically do on a production server) and that's what we need to do also on linux.
The most simple approach is to use docker in such a way we have a full pre-fab AspNet 5 environment:
```language-bash
sudo docker run -it -p 5000:5000 microsoft/aspnet /bin/bash 
```
Inside the container, do a git clone of the 
```language-bash
dnx Kestrel
```
## Conclustion
I love it !
Currently, I'm using in the sample the EntityFramework In Memory database provider. I'm pretty sure there will be exquisite support for SqlServer but trying the sqlite provider was without success. 
I hope one day I will combine both .Net and NodeJs apps on my favorite server environment Linux and simply chose between .Net or Node depending of typical needs and the required functionality.
