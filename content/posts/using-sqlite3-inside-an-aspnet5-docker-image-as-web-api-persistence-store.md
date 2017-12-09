+++
date = "2015-09-11T07:24:39+00:00"
draft = false
title = "Using sqlite3 inside an aspnet5 docker image as web api persistence store."
slug = "using-sqlite3-inside-an-aspnet5-docker-image-as-web-api-persistence-store"

+++

## introduction
In my previous [post](http://blog.opinionatedapps.com/running-a-asp-net-5-web-api-side-by-side-on-windows-and-linux/) I used an in memory database as "persistence" for the web api service. That's of course not a viable option. In this article I'll explain how you can update the aspnet docker image in such a way you can use sqlite3. But let's first check why you would want to use sqlite?

## why sqlite?
For occasion you don't want or need a full sql server. So, for demo or test purposes.
## create and tweak docker image
```language-bash
sudo docker run -it -p 5000:5000 microsoft/aspnet /bin/bash  
```
Inside the docker image :
```language-bash
apt-get update
apt-get upgrade
apt-get install git nano
mkdir /github
cd github
git clone https://github.com/paulvanbladel/AspNet5-Web-Api-Sample.git
```
Now basically, it would be only a matter of installing now sqlite3 via apt-get:
```language-bash
apt-get install  sqlite3 libsqlite3-dev
```
but that won't work because it will install an outdated version of sqlite3.
## Update sources.list
```language-bash
nano /etc/apt/sources.list
```
and add following line:
```language-bash
deb http://ftp.us.debian.org/debian jessie main
```
run now
```language-bash
apt-get update
apt-get install  sqlite3 libsqlite3-dev
```
## check if you have a version higher than 3.7.15.
```language-bash
root@e47451784bb8:/etc/apt# sqlite3
SQLite version 3.8.7.1 2014-10-29 13:59:56
Enter ".help" for usage hints.
```
You can press .exit for leaving the sqlit3 prompt.

## provision the web api project
```language-bash
cd /github/AspNet5-Web-Api-Sample/src/Web-Api-Sample
dnvm use 1.0.0-beta7
export MONO_THREADS_PER_CPU=2000
dnu restore
dnu build
```
The last line (the build command) should reveal:
```language-bash
Build succeeded.
    0 Warning(s)
    0 Error(s)
```

If you are getting errors, ... Houston you have a problem.
## Run the web api project
```language-bash
dnx Kestrel
```
Browse now to http://localhost:5000/api/customer and you will see some data.
You can generate additional dummy data by calling http://localhost:5000/api/customer/mimicCustomerImport/100.
By doing so an additional 100 rows are created in the database.