+++
date = "2015-01-15T14:38:18+00:00"
draft = false
title = "Adopting a lean web app server infrastructure based on nginx (part 2/3)"
slug = "adopting-a-lean-web-app-server-infrastructure-based-on-nginx-part-2"

+++

##Enjoy the mechanical sympathy of nginx?
For me, the most important aspect for selecting [nginx](http://wiki.nginx.org/Main) (pronounce Engine-X) is the fact that it basically uses an **event-driven architecture for handling request in an asynchronous matter**, much like nodeJs is working. In other words, it doesn't rely on threads to handle requests.  Since we'll jump soon on the nodeJs bandwagon, you'll start to like this type of [mechanical sympathy](http://martinfowler.com/articles/lmax.html).
All this makes nginx very lightweight and super fast.
Furthermore, nginx has a significant market share next to Apache and IIS.

##Install git and nodeJs on Ubuntu
Ubuntu uses the[ APT package manager](https://help.ubuntu.com/community/AptGet/Howto).
Let's start with Git:
<pre><code class="language-bash" >sudo apt-get install git</code></pre>
and then NodeJs:
Update march 2015: there can be occasions where you want to follow [this](https://www.digitalocean.com/community/tutorials/how-to-install-an-upstream-version-of-node-js-on-ubuntu-12-04) approach for installing nodeJs.

<pre><code class="language-bash">sudo apt-get install nodejs</code></pre>
we'll need also npm, the node package manager:
<pre><code class="language-bash" >sudo apt-get install npm</code></pre>
On Ubuntu (and Debian in general), the name of the Node.js interpreter is changed to NodeJS (instead of Node). It's best to change this back to node by creating a symlink, this can be done by installing following package:
<pre><code class="language-bash" >sudo apt-get install nodejs-legacy</code></pre>
A more experienced linux user will install them all with a one liner:
<pre><code class="language-bash">sudo apt-get install git nodejs npm nodejs-legacy</code></pre>

##Install nginx
<pre><code class="language-bash">sudo apt-get install nginx</code></pre>

##Check if the server is working
![](http://blog.opinionatedapps.com/content/images/2015/01/nginx2_1.png)
Obviously, you can configure your DNS in such a way the IP address points to www.MySite.com.
We'll need to do this anyway in later examples where we create a static site in a sub domain.
Bottomline so far is that installing a web server software on Ubuntu is really a piece of cake and a matter of 30 seconds. That's the way we want it.

##Show me now a nice UI for configuring Nginx
In case you are expecting now to be able to use a fancy graphical user interface for configuring everything, I need to disappoint you. Nginx (and yeah, everything else in linux) needs to be configured via config files from the command prompt. 
That's really a huge difference with the windows world where a lot of energy is spent for building good looking interfaces. Anyhow something like IIS stores it's information that is entered via the UI also in xml config files, which you only need if you messed things completely up in the UI interface. In that scenario, you can pray and hope that you understand everything what you are doing. In the linux world you need that knowledge from day 1, that's the difference, but you are in case of trouble better prepared.
On the other hand, what is important is the amount of lines that need to be 'written' for configuring an additional site/application. 
Finaly, these types of config files let themselves integrate elegantly in an auomated software factory approach.

##Let's get some basic understanding of the nginx configuration files
Let's inspect the base configuration file of nginx located in  /etc/nginx/nginx.conf by typing:
<pre><code class="language-bash">cat /etc/nginx/nginx.conf</code></pre>
This will give us following content:
<pre style="background-color:lightgrey"><code >
user www-data;
worker_processes 4;
pid /run/nginx.pid;

events {
        worker_connections 768;
        # multi_accept on;
}

http {
        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##

        gzip on;
        gzip_disable "msie6";

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # nginx-naxsi config
        ##
        # Uncomment it if you installed nginx-naxsi
        ##

        #include /etc/nginx/naxsi_core.rules;

        ##
        # nginx-passenger config
        ##
        # Uncomment it if you installed nginx-passenger
        ##

        #passenger_root /usr;
        #passenger_ruby /usr/bin/ruby;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}
</code></pre>
The file is quite long, but that's because it contains a lot of examples (commented out).
What we learn here is that:

* sites will run under a specific user named : www-data. That will be important when configuring permissions.
* there are access and error logs in /var/log/nginx/access.log en error.log. It's great that we can find all loggings in a central place /var/log/...
* right now, the most important lines in this config file are at the end of the http block: <pre><code class="language-bash">		include /etc/nginx/conf.d/*.conf;
		include /etc/nginx/sites-enabled/*;
</code></pre>       

The meaning of the include statements is quite obvious: all config files in these 2 folders will be incorporated in the master config file. That gives a very simple and elegant way to configure a whole server. So far, I don't see only a need for using the sites-enabled folder. But you need to use it in combination with the sites-available folder on the same level as the sites-enabled folder.

##What's the difference between the folder sites-enabled and sites-available?

When you do: 
<pre><code class="language-bash">ls /etc/nginx
</code></pre>
you will notice there is also a sites-available folder.

The whole idea of these folders is to be able 

* to make a configuration of a site but not activate it yet or;
* to temporary disable a site.

How does this work?  Type:
<pre><code class="language-bash">ls /etc/nginx/sites-available/ -al/nginx
</code></pre>

<pre style="background-color:lightgrey"><code>
total 12
drwxr-xr-x 2 root root 4096 Jan 15 09:24 .
drwxr-xr-x 5 root root 4096 Jan 15 09:24 ..
-rw-r--r-- 1 root root 2617 Oct 22 12:27 default
</code></pre>

Nothing special so far, the folder contains a regular file called default, our default config file.

Do now the same for the sites-enabled folder:
<pre><code class="language-bash">ls /etc/nginx/sites-enabled/ -al
</code></pre>

<pre style="background-color:lightgrey"><code >
total 8
drwxr-xr-x 2 root root 4096 Jan 15 09:24 .
drwxr-xr-x 5 root root 4096 Jan 15 09:24 ..
lrwxrwxrwx 1 root root   34 Jan 15 09:24 default -> /etc/nginx/sites-available/default

</code></pre>

You can see that the default file has here a special 'l' attribute, which stand for link. This file is a symbolic link to the path indicated at the end of the line. In windows terminology: it's a shortcut !
So, the whole idea is that the folder sites-available contains the real files and a link inside the sites-enabled folder is constructed via following command:
<pre><code class="language-bash">
ln -s /etc/nginx/sites-available/example.com.conf /etc/nginx/sites-enabled/example.com.conf
</code></pre>

##Conclusion
In my humble opinion: simple and straight-forward.
In a next episode, we'll see how to create static sites in sub domains and sub folders.









