+++
date = "2015-01-20T15:12:11+00:00"
draft = false
title = "Running a NodeJs app on nginx (1/2)"
slug = "running-a-nodejs-app-on-nginx-part-1"

+++

##Introduction
In the previous post we did setup a nginx web server for serving static pages.
In this post, we will host a NodeJs application and make sure it will survice application crashes.

##The sample NodeJs application
We'll focus on how nginx handles NodeJs, so we just need a dummy app but we want that the app logs both normal and failure behavior.
So, let's create a NodeJs application under /usr/share/nginx/app1. We'll create the app today directy on our server. It's a good habit to first create a package.json file by typing in the app1 folder:
<pre><code class='language-bash'>npm init
</code></pre>
This will guide you through a kind of wizard resulting in a package.json file:
<pre><code class='language-markup'> "name": {
  "app1",
  "version": "0.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "node index",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}

</code></pre>
Note that we slightly adapted the package.json in such way there is an npm start verb. We did this by adding under "scripts": "start": "node index".

```language-javascript
var http = require('http');
http.createServer(function (req, res) {
    var input = req.url.toString();
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('thanks for ' + input + ' on ' + new Date().toISOString() + '\n');
    if(input=='/warning'){
        console.info("a warning occured on " + new Date().toISOString());
    }
    if (input=='/crash'){
        console.error("a crash occured on " + new Date().toISOString());
        throw "serious error";
    }
}).listen(1337, '127.0.0.1');

console.log('Server running at http://127.0.0.1:1337/' + ' ' + new Date().toISOString());
```
 
As you can see, not really a large scale business application. The app simply logs the incoming url and covers 2 specific cases:

* when the url matches exactly '/warning' a specific logging occurs and
* when the url matches exactly '/crash', we raise an error and make an logging in the error log.

This provides us immediately two important scenarios:

* we are making use of the **relative url** in this app (the input variable). So, when the user provides as url: app1.opinionatedapps.com/mytext, the input variable will be 'mytext". But what happens when we deploy our application not directly on a subdomain but in a subfolder. Well, a requirement will be that the application should not be aware of that. So, our nginx configuration should make sure that demosite1.opinionatedapps.com/app1/mytext gives as url also "mytext" and not  "/app1/mytext".
* when the app crashes, **we need something to restart and recover automatically**. Nginx will not do this for us. That's a big difference with a web server like IIS running Asp.Net apps. In the IIS scenario, both the runtime (Asp.Net) and the server (IIS) are quite intimitely integrated. That's not the case with nginx which doesn't know anything at all about about NodeJs. At first glance, this feels like a step backwards when it comes to comfort but in a way it's a much cleaner seperation of concerns and it reflex the Unix philosophy of components doing simply one thing but doing it really good.
Ok, let's first test if the app runs by typing : 
<pre><code class='language-bash'>cd /usr/share/nginx/app1
npm start
</code></pre>

You should see now in the console something like : Server running at http://127 ...
Obviously, so far our app runs also as a backend process on our server, we need to configure nginx to expose the application behind port 80. That's what we'll do next.
## configure nginx for the NodeJs app
We'll need to update the file demosite1.opinionatedapps.com inside the /etc/nginx/sites-enabled folder as follows:
<pre><code class='language-bash'>cd /usr/share/nginx/app1
server {
listen 80;
server_name demosite1.opinionatedapps.com;
location /{
        root /usr/share/nginx/testsite;
        index index.html;
}
location /sub1{
        alias /usr/share/nginx/sub1/;
        index index.html;
}
location ^~ /app1{
        proxy_pass http://localhost:1337;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-NginX-Proxy true;
        proxy_redirect off;
}
}
</code></pre>

The above assures that nginx acts as kind of reverse proxy and forwards all requests to /app1 to port 1337 where nodeJs is running our app1.
Let's try now from the browser:

http://demosite1.opinionatedapps.com/app1/HiThere 
which gives as output:
<pre><code>"thanks for /app1/HiThere on 2015-01-18T10:20:59.587Z" 
</code></pre>
and that's not what we want. We don't want to see 'app1' in the url, but we want a path relative to the application root inside the app1 folder !
We can accomplish this by adding following line inside the location block of app1:
<pre><code class='language-bash'>rewrite ^/app1/(.*)$ /$1 break;
</code></pre>
##Force an app crash now
http://demosite1.opinionatedapps.com/app1/warning is triggering a warning message, which we can see in the output window of the NodeJs app. That's good. But what happens if we trigger an app crash by browsing to: 

<pre><code>
a crash occured on 2015-01-18T10:58:22.212Z
/usr/share/nginx/app1/index.js:11
        throw "serious error";
        ^
serious error
npm ERR! app1@0.0.0 start: `node index`
npm ERR! Exit status 8
npm ERR!
npm ERR! Failed at the app1@0.0.0 start script.
npm ERR! This is most likely a problem with the app1 package,
npm ERR! not with npm itself.
npm ERR! Tell the author that this fails on your system:
npm ERR!     node index
npm ERR! You can get their info via:
npm ERR!     npm owner ls app1
npm ERR! There is likely additional logging output above.
npm ERR! System Linux 3.16.0-28-generic
npm ERR! command "/usr/bin/nodejs" "/usr/bin/npm" "start"
npm ERR! cwd /usr/share/nginx/app1
npm ERR! node -v v0.10.25
npm ERR! npm -v 1.4.21
npm ERR! code ELIFECYCLE
npm WARN This failure might be due to the use of legacy binary "node"
npm WARN For further explanations, please read
/usr/share/doc/nodejs/README.Debian

npm ERR!
npm ERR! Additional logging details can be found in:
npm ERR!     /usr/share/nginx/app1/npm-debug.log
npm ERR! not ok code 0
</code></pre>

That's all as expected, but we have a serious problem now, the app stopped completely.
We'll cover that scenario in the next episode.

