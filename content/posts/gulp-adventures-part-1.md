+++
date = "2015-03-12T10:07:32+00:00"
draft = false
title = "Gulp adventures (part 1/n)"
slug = "gulp-adventures-part-1"

+++

##Introduction
I'm learning Gulp and sharing here my progressive insights. I'm a big fan of a decent continuous integration approach and to foresee as much automation as possible in your application development workflow. All that makes that your can put on your thinking caps for the right purposes, namely thinking about the software your are building. Furthermore an approach of "Deploy early and deploy often" will enhance the relationship with your stakeholders. 

##Why gulp?
Indeed, there is also the slightly older "grunt" task runner. I prefer Gulp because it's code over configuration, it's just nodeJs code and I like the stream based (piping) approach.
Another advantage is that the base API has only four command: 
* gulp.task : for defining a task
* gulp.src : for reading files
* gulp.dest : for writing files
* gulp.watch : watching files and acting upon

In short, I love the simplicity and power of Gulp.

##Types of tasks
In a typical single page web application based on an angular front-end and a nodeJs back-end you will want to do following tasks:

* development environment: startup nodeJs back-end and serve the static files
* index.html injection 
* testing 
* code analysis 
* Minification
* concatenation 
* vendor prefixes
* Less compilation
* ...

##Let's take a developer friendly approach
How many times did you clone a repository from github or elsewhere and did you get irritated by the amount of time you needed to figure out how to simply start up the project. Things should start out of the box, right? The same goes for .Net projects in visual studio, if you can't simply F5 them, there is something fundamentally wrong.
The 'workflow' I have in mind boils down to:

* we presume you have correctly installed the **prereq tools** listed in following section
* **npm install** should install the necessary node packages and **trigger bower install automatically**
* **the default gulp task** should  **spin up everything else** in such a way both the back-end is started and a browser is opened with the front-end served correctly.
* once running, the above default gulp task should be able to continue running **without interruption**
* the default gulp task should **manipulate your source code as little as possible**.

##Prereq tools
* NodeJs
* Npm install bower -g
* Npm install gulp -g

##What's next
In the next article, we'll focus on 

* the development server, 
* nodeMon for keeping nodeJs up and running and 
* the injection of bower css/js and custom css/js file reference injection in the index.html of your single page application.

Stay tuned.
