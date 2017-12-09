+++
date = "2015-03-12T12:47:55+00:00"
draft = false
title = "Gulp adventures (part 2/n)"
slug = "gulp-adventures-part-2"

+++

## Project structure
Take a look at following project structure:
![](/content/images/2015/03/solutionstructure-1.png)
There is the client and server folder in the root. Some developers surround these with a additional src folder. I don't see any direct benefit in doing that.
Another observation is that the bower_components are stored under client/app. The reason is that bower components are really tied to the client, they are never used from the server project. This in contrast with the npm modules, which are used both by the server as well as the client. For the client, they are mostly used as development dependencies.
So far my project doesn't contain any real content. Only an index.html, an app.js and app.css and server side a server.js. But it's enough material for the purpose of this post: **setting up an comfortable development experience.**

##Steps from the perspective of someone not familiar with your project.

Recall from the previous post, the 3 prereqs:
* nodejs must be installed
* npm install -g bower
* npm install -g gulp
That looks like an acceptable assumption, no? NodeJs, bower and gulp must be globally present.

1. git clone
2. npm install
3. gulp

Less is more, you know.

##What does really matter during development?
Speed & transparancy.

What I mean with speed is that the gulp task for setting up your development server should not shuffle around with your source files. Not only because it slows down things, but also because the real power of modern web apps (simply based on html, js and css) is precisely it's transparancy.  Things can be ran without complex compilation processes as it is the case in het Java and .Net world. 
There is nothing wrong with a kind of 'build' process when it comes to preparing the project for a deployment, but during the development phase transparancy is key, and your project should simply run directly from the sources, with the simple exception for:

* the index.html file: bower dependencies both for css and js as well as project specific css/js should be injected automatically.
* Less compilation (not covered in this post)

##Split gulp processing from gulp configuration.
A good practice (which I learned from **John Papa**) is to use apart from the gulpfile also a js file called gulp.config.js and which contains the 'moving pieces', the file paths, ports and stuff like that. This approach favors reuse of the same gulpfile between projects.
So a boiler plate gulp.config.js file can look as follows:
```language-javascript
module.exports = function () {
    var path = require('path');
    var root = './';
    var client = path.join(root, 'client');
    var server = path.join(root, 'server');
    var clientApp = path.join(client, 'app');
    var serverApp = server;

    var config = {
        client : client,
        server: server,
        clientApp: clientApp,
        clientIndex: path.join(client, 'index.html'),

        serverIndex: path.join(serverApp, 'server.js'),
        js: [
            path.join(clientApp, 'js/*.js'),
            path.join(clientApp, 'modules/*.js'),
            path.join(clientApp, 'modules/**/*.js')
        ],
        css: [
            path.join(clientApp, 'css/**/*.css'),
            path.join(clientApp, 'modules/**/*.css')
        ],
        optimized: {
            app: 'app.js',
            lib: 'lib.js'
        },
        build: path.join(root, 'build'),
        defaultNodePort: 4000,
        defaultWebServerPort: 7000
    };
    return config;
};


```
##The index task
So, currently the only task where we deviate a bit from the above formulated 'transparancy' principle but with good reasons. How many times did you forget to include a script or stylesheet reference in index.html?

```language-javascript
var gulp = require('gulp');
var bowerFiles = require('main-bower-files');
var config = require('./gulp.config')();
var plugin = require('gulp-load-plugins')({lazy: true});
gulp.task('index', function () {
    return gulp.src(config.clientIndex)
        .pipe(plugin.inject(gulp.src(bowerFiles(), {read: false}),
        {name: 'bower', relative: true}))
        .pipe(plugin.inject(gulp.src(config.js, {read: false}), {relative: true}))
        .pipe(plugin.inject(gulp.src(config.css, {read: false}), {relative: true}))
        .pipe(gulp.dest(config.clientApp));
});

```
So, we require-in the config file and use it directly in the index task.
The gulp plugin gulp-load-plugins fascilates loading of plugins in the sense that you don't need to require them in explicitely.
Consult [the gulp documentation](https://github.com/gulpjs/gulp/blob/master/docs/API.md) for having a good understanding of the pipe mechanism.

Our index.html should contain some commented out placeholders:

```language-markup
<!DOCTYPE html>
<html ng-app="app">
<head>
    <meta charset="utf-8"/>
    <meta http-equiv="X-UA-Compatible" content="IE=edge, chrome=1"/>
    <meta name="viewport"
          content="width=device-width, initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no"/>
    <title ng-bind="title"></title>
    <base href="/">
    <!-- bower:css -->
    <!-- endinject -->
    <!-- inject:css -->
     <!-- endinject -->
</head>
<body>
<div>
    <p>hi there  </p>
    <div id='result'></div>
</div>
<!-- bower:js -->
<!-- endinject -->
<!-- inject:js -->

<!-- endinject -->
</body>
</html>
```
so when we run now gulp index, we get:
```language-javascript
<!DOCTYPE html>
<html ng-app="app">
<head>
    <meta charset="utf-8"/>
    <meta http-equiv="X-UA-Compatible" content="IE=edge, chrome=1"/>
    <meta name="viewport"
          content="width=device-width, initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no"/>
    <title ng-bind="title"></title>
    <base href="/">
    <!-- bower:css -->
    <link rel="stylesheet" href="bower_components/bootstrap/dist/css/bootstrap.css">
    <!-- endinject -->
    <!-- inject:css -->
     <link rel="stylesheet" href="css/app.css">
     <!-- endinject -->
</head>
<body>
<div>
    <p>hi there  </p>
    <div id='result'></div>
</div>
<!-- bower:js -->
<script src="bower_components/jquery/dist/jquery.js"></script>
<script src="bower_components/bootstrap/dist/js/bootstrap.js"></script>
<script src="bower_components/angular/angular.js"></script>
<script src="bower_components/angular-ui-router/release/angular-ui-router.js"></script>
<script src="bower_components/angular-bootstrap/ui-bootstrap-tpls.js"></script>
<script src="bower_components/underscore/underscore.js"></script>
<!-- endinject -->
<!-- inject:js -->

<script src="modules/app.js"></script>
<!-- endinject -->
</body>
</html>
```
##making the index task dependent on bower install
Wouldn't it be nice if the index task whould simply run every time we add a new bower dependency?
That can be achieved with following config in the .bowerrc file:
```language-javascript
{
  "directory": "client/app/bower_components",
  "scripts": {
    "postinstall": "gulp index"
  }
}
```
Now, let's go one step further and make bower install depending on npm install. This can be done in package.json:
```language-javascript
{
  "name": "GulpAdventures",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "init": "npm install",
    "install": "bower install"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "gulp": "^3.8.11",
    "gulp-inject": "^1.2.0",
    "gulp-load-plugins": "^0.8.1",
    "gulp-nodemon": "^1.0.5",
    "gulp-task-listing": "^1.0.0",
    "gulp-util": "^3.0.4",
    "main-bower-files": "^2.5.0"
  },
  "dependencies": {
    "express": "^4.12.2"
  }
}
```
Notice in the script section the bower install command.
So we have now a nice **chain of dependencies**:
1. Npm install triggers bower install
2. Bower install triggers gulp index and update the index files with the latest bower material.

##NodeMon will monitor our back-end files
```language-javascript
gulp.task('nodejs', function () {
    var nodeOptions = {
        script: config.serverIndex,
        delayTime: 1,
        env: {
            'PORT': nodeJsPort
        },
        watch: [config.server]
    };
    nodemon(nodeOptions)
        .on('change', function () {
            log('nodemon detected change...!')
        })
        .on('restart', function () {
            log('node application is restarted!')
        })
});
////////////////////////////////////////////////////////////////////
function log(msg) {
    $.util.log( $.util.colors.green( msg));

}
```


##Don't let NodeJs serve the static files
It's a good practice to NOT let NodeJs serve your static files. I know, it's the simplest solution  to use nodeJs for this, but in production, it's much faster if the webserver takes care of this.
Therefor, it's better to stay as close to this in the development environment as well.
##The development web server
```language-javascript
gulp.task('webserver', ['nodejs'], function () {
    browserSync({
        browser: "chrome",
        port: webServerPort,
        server: {
            baseDir: config.client
        },
        files: [
            config.clientApp + '/**/*.*',
            config.clientIndex
        ],
        middleware: function (req, res, next) {
            var url = req.url;

            if (url.substring(0, 5) === "/api/") {
                proxy.web(req, res, {target: 'http://localhost:' + nodeJsPort});
            } else {
                next();
            }
        }
    });
});
```
Notice we are proxy-ing are nodeJs app running on the port taken from the config file in such a way we have not to deal with  [CORS](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing) matters. All request starting with /api/ will not be handled by the webserver but will be forwarded to the NodeJs app running on the nodeJs port. I think this is a nicer separation of concerns: a web server is meant for serving files, nodeJS is just an application listen on a particular tcp socket. In an later post I will show how we can easily do this in production with the Nginx (web) proxy server.
You probably noticed as well that our webserver task calls first the nodejs subtask. 
when running the webserver task, you can start programming without having the reload anything at all. A server side changed is handled by nodemon and every change in the client project will be immediately reflected in the browser. Cool !

To finish off for today, let's created 2 additional tasks. One for our 'default' task and one for a help task.

##A developer centric default task
In order to be able to simply run gulp for starting the webserver we need to add:
```language-javascript
var plugin = require('gulp-load-plugins')({lazy: true});
gulp.task('default', ['webserver']);
gulp.task('help', plugin.taskListing);
```
##What's next?
We'll add a build task for preparing the sources for deployment.
You can find the sources of the above on [github](https://github.com/paulvanbladel/GulpAdventures).


