+++
date = "2015-01-22T21:33:07+00:00"
draft = false
title = "Running a NodeJs app on nginx (2/2)"
slug = "running-a-nodejs-app-on-nginx-part-2"

+++

##Introduction
In the first episode, we did setup nginx for hosting a nodeJs app and applied an additional rewrite rule. In this post we'll provide some additional rebustness.
What we are missing, is a kind of process which monitors our app and which assures that the app stays up and running. Furthermore the process should also make sure that the app is started when the machine reboots.
So, you could call this process manager a kind of supervisor and indeed, that's what we need: supervisor.
##Introducing supervisor
Let's rely on apt-get to install supervisor: 
<pre><code class='language-bash'>sudo apt-get install supervisor
</code></pre>
When you install supervisor it will start immedidately and as a bonus it will also start automatically when the machine boots. Great, that went easy !

##Configure supervisor to monitor our NodeJs app
In the directory /etc/supervisor you will find the general supervisor configuration file (supvisor.conf). We don't need to touch that one.
We can provide inside the folder /etc/supervisor/config.d a dedicated config files for each process that we want to monitor.
So, create file called app1.conf and provide following content:

<pre><code class='language-bash'>
[program:app1]
command=npm start
directory=/usr/share/nginx/app1
autostart=true
autorestart=true
stderr_logfile=/var/log/app1.err.log
stdout_logfile=/var/log/app1.out.log
</code></pre>

We then need to make sure that this new file is executable and reread the supervisor configuration.
Following command will do this:
<pre><code class='language-bash'>sudo chmod +x app1.conf
sudo supervisorctl reread
sudo supervisorctl update
</code></pre>

##Check if everything is working as expected
* Try now to reboot the server and check the app is automatically running.
* Try also to let the app crash by browsing to 
http://demosite1.opinionatedapps.com/app1/crash
* Try also to trigger some logging in the standard log file.Consult the log files: /var/log/app1.err.log and /var/log/app1.out.log will contain the app's loggings.

##Conclusion
Indeed, a bit more work than some clicking around when you come from IIS and indeed you need to be precise but it's all rock solid and it's perfectly scriptable when you think already in terms of deployment scripts.
What we didn't cover so far are the security aspects (file permissions and user management). That will be the subject of a later post.

