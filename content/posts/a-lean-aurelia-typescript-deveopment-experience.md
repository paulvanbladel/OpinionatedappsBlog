+++
date = "2015-08-08T10:59:54+00:00"
draft = false
title = "A lean aurelia typescript development experience"
slug = "a-lean-aurelia-typescript-deveopment-experience"

+++

##Introduction
I have seen many attempts to introduce typescript into the aurelia development pipeline. Usually, what they  have in common, is that they are are all quite involved and complex.
I tried to work the other way around, based on the new jspm beta, and I'm proposing here an extremely simplified setup for working with typescript in aurelia. Note that I'm only focussing on a development experience. Hopefully, this example is interesting enough to trigger a discussion towards a better aurelia typescript experience.

##show me the sources
```bash
git clone https://github.com/paulvanbladel/aurelia-typescript-lean-dev.git
```
Please follow the installation instructions in the [Readme](https://github.com/paulvanbladel/aurelia-typescript-lean-dev/blob/master/README.md).

##what are the most striking differences?
We are using the new jspm beta here.
We use typescript files and let systemJs compile the sources in the browser. So basically, **at least for pure development**, we no longer need gulp. That's why I deleted to the complete build folder. Note that this done only to make my point clear, obviously, gulp build tools are still needed for other tasks. Instead of the gulp serve task, I'm simply using live-server (which can be installed by npm install live-server -g).

We no longer need babel, so I removed it from the jspm/npm.

I'm using the aurelia-skeleton sample and renamed all .js files in the src folder to .ts.
Furthermore I did some small updates so that we are also using some typescript specific functionality.
In the following sample we use http in a typed manner:
```language-javascript
constructor(http: HttpClient){
    http.configure(config => {
      config
        .useStandardConfiguration()
        .withBaseUrl('https://api.github.com/');
    });

    this.http = http;
  }
```
##what's different in config.js ?
Apart from changing the .js files to .ts in the src folder, we need to make a small update in config.js but the changes are very limited.

We need to tell systemJs to use typescript as transpiler and need to adapt slightly the path block:

```language-javascript
System.config({
  "defaultJSExtensions": true,
  "transpiler": "typescript",
  "paths": {
    "*": "src/*",
    "src": "src",
    "github:*": "jspm_packages/github/*",
    "npm:*": "jspm_packages/npm/*"
  },
  "packages": {
    "src": {
      "defaultExtension": "ts"
    }
  }
});

System.config({
  "map": {
  ...
  ...  
```
Note, that all babel related config is gone !
## Debug experience
In order to start the aurelia app, simply start live-server.
Typescript files are now compiled on the fly. Your browser's F12 tools wil reveal the compiled files:
![](/content/images/2015/08/F12.png)

##Why am I showing this?
I not only love typescript and I believe typescript is  simply a necessity when it comes to large scale javascript applications. Without typescript things become unmaintainable.
But we need a better typescript development experience. This example is probably going a bridge too far my completely eliminating the build process, but at least it's simple and straight-forward. So, try it out and le me know what's missing...


