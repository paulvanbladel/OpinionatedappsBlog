+++
date = "2015-01-15T19:58:06+00:00"
draft = false
title = "Adopting a lean web app server infrastructure based on nginx (part 3/3)"
slug = "adopting-a-lean-web-app-server-infrastructure-based-on-nginx-part-3"

+++

##Introduction
In this post, we'll create two types of static sites:

* in a sub domain : e.g. demosite1.opinionatedapps.com
* in a sub folder: e.g. demosite1.opinionatedapps.com/sub1
##Create a static site in a sub domain
First let's go to our DNS (in my case NameCheap.com) and add a new sub domain called 'demosite1':
![](http://blog.opinionatedapps.com/content/images/2015/01/nginx2_2.png)

Let's create a config file for this sub domain site in /etc/nginx/sites-available:
<pre><code class="language-bash">nano /etc/nginx/sites-available/demosite1.opinionatedapps.com</code></pre>
<pre><code>server {
listen 80;
server_name demosite1.opinionatedapps.com;
location /{
        root /usr/share/nginx/demosite1;
        index index.html;
        }
}
</code></pre>
When doing a change to a nginx config file, you need to do a reload:
<pre><code class="language-bash">sudo nginx -s reload
</code></pre>
And create a symbolic link towards the sites-enabled directory:
<pre><code class="language-bash">ln -s /etc/nginx/sites-available/demosite1.opinionatedapps.com /etc/nginx/sites-enabled/demosite1.opinionatedapps.com</code></pre>
When you browse now to demosite1.opinionatedapps.com you will get a 404, because the /usr/share/nginx/demosite1 folder has no content yet. Therefor, create a dummy index.html file in that folder
<pre><code class="language-bash">
sudo nano /usr/share/nginx/demosite1/index.html
</code></pre>
with following content:
```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
</head>
<body>
<h1>Welcome to demo site 1!</h1>
</body>
</html>`
```

##Create a static site in a sub folder of a sub domain
Now imagine, that we want also to have a dedicated site in a sub folder of our newly created sub domain (e.g. demosite1.opinionatedapps.com/sub1)

That's just a matter of updating /etc/nginx/sites-available/demosite1.opinionatedapps.com with:
<pre><code>
server {
listen 80;
server_name testsite.opinionatedapps.com;

location /{
        root /usr/share/nginx/demosite1;
        index index.html;
}

location /sub1{
        alias /usr/share/nginx/sub1/;
        index index.html;
}
}
</code></pre>
We did again a change to a nginx config file, so we need to do a reload:
<pre><code class="language-bash">sudo nginx -s reload
</code></pre>
Create similarly a dummy html file under /usr/share/nginx/sub1.

##Conclusion
Pretty simple. Obviously this file based approach can be easily integrated in a software factory  where we will typically generate the appropriate config files programmaticaly.
In a later episode, well leave the safe path of static sites and jump on the web application bandwagon.
Stay tuned.
