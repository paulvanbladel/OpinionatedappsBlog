+++
date = "2015-08-11T15:51:30+00:00"
draft = false
title = "an uber simple gulp typescript workflow for Aurelia"
slug = "an-uber-simple-gulp-typescript-workflow-for-aurelia"

+++

## Introduction
In the previous  [article](http://blog.opinionatedapps.com/a-lean-aurelia-typescript-deveopment-experience/) we saw how we could let the browser do the typescript compilation.
I had today a very interesting chat with [Louis Lewis](https://github.com/louislewis2) and he provided me the missing pieces for transforming the skeleton-navigation app with a minimal effort to  a full blown typescript app while preserving the systemJs module syntax.

## show me the code
[Here we go.](https://github.com/paulvanbladel/aurelia-gulp-typescript)

## the gulp-typescript compile task
the most important difference is of course is kicking out babel as transpiler and using typescript instead:

```language-javascript
var ts = require('gulp-typescript');
var tsProject = ts.createProject('tsconfig.json');
gulp.task('build-system', function () {
  return gulp.src(["jspm_packages/**/*.ts", "typings/**/*.d.ts", paths.source])
    .pipe(plumber())
    .pipe(ts(tsProject))
    .pipe(gulp.dest(paths.output));
});
```

So, a kind of virtual typeScript project is created in memory and compiled. The virtual project is composed of :

* the .d.ts files in the folder jspm_packages
* application specific typing files in the typing folder
* and our actual application typescript files.

The basic folder structure of your project is very similar to the normal skeleton-navigation app except it contains also the typings folder.
The virtual typescript project is based on some settings stored in the root of the project and called tsconfig.json:
```language-javascript
{
  "compilerOptions": {
     "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "noImplicitAny": false,
    "noEmitOnError": true,
    "removeComments": false,
    "noExternalResolve": true,
    "declarationFiles": false,
    "target": "es5",
    "module": "commonjs"
  }
}
```
The nice thing about this tsconfig.json approach is that it is universal. Modern editors like sublime text recoginze that file and will adapt the compilation behavior of the editor accordingly. That said, I prefer the gulp-typescript approach above smart editors. With the gulp solution, you work really editor independent !

## let's cope with a small glitch
You might have noticed that currently the project is not a perfect copy of the current skeleton-navigation app which uses the fetch-client instead of the http-client.
Today, 11 augustu 2015, the .d.ts file for the fetch-client contains a bug. That's why I took the sample ts files from [Louis Lewis](https://github.com/louislewis2), who solved this elegantly by switching to the http-client.

Of course, we could try to fix that bug in the fetch-client's typings file, but for now, let's simply delete the entry in the jspm_packages folder: 
![](/content/images/2015/08/fetchclient.png)

Afterwards, you can simply do:
```language-bash
gulp watch
```
and you have working aurelia typescript project. A loud shout-out to [Louis Lewis](https://github.com/louislewis2) for his inspiration !
