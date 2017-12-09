+++
date = "2016-10-03T12:20:48+00:00"
draft = false
title = "Get a CLI inside an aurelia SPA in minutes with aurelia-app-cli"
slug = "aurelia-app-cli-in-minutes"

+++

## introduction
There is a big revival of Command Line Interfaces (CLI). Windows 10 has today an ubuntu bash shell on board, many application development frameworks promote their own CLI.

Also inside a single page application (a SPA), a CLI can be very useful.
In this post we we look into the functionality of the aurelia-app-cli plugin. In a later post I'll dive deeper into the internals of the plugin.

## why does my SPA needs a CLI ?
A CLI allows a user to execute various tasks when using a CLI.
### some tasks can be executed faster via a CLI
Obviously, we build fancy user interfaces because we believe that the typical user experience of a SPA will do things better. Nonetheless for certain tasks a CLI is simply faster than even the most fine-tuned UI. A canonical example is unlocking a user by an admin. Typing in the CLI "unlock paulvanbladel" is indeed faster than browsing through various screen just to get the same effect. An admin who needs to do this tasks 100 times a day, will be more happy with a CLI. 

### a CLI allows early preview and prototyping of functionality that eventually will have a full UI

Often, new functionality has both a client and server part. When the client side design is demanding, a CLI allows that the server part can be used already and tested in an earlier fase (e.g. for a limited set of users)

## features
Go to the [sample application](https://paulvanbladel.github.io/aurelia-app-cli-sample/#/welcome) so that you can verify following features.

#### a CLI should look like a CLI
Something like this, right?
Sure, CSS I stole somewhere, monospace fonts and other fancy stuff providing an great CLI user experience.

![aurelia-app-cli](/content/images/2016/10/app-cli.png)

#### a CLI should be self explaining and provide help
Go to the [sample application](https://paulvanbladel.github.io/aurelia-app-cli-sample/#/welcome) and type in the CLI: help.
You'll get a list of command that can executed. The commands CLS, Help and Welcome are part of the aurelia-app-cli plugin itself. The other commands are 'project specific' to this sample application. The project specific commands are automatically merged with the built-in commands. Which brings us to the next feature:
#### the CLI's command set should be extensible in a simple manner
We'll explain that in deeper detail in a next blog post.
#### a CLI should keep track of a command history
Go to the [sample application](https://paulvanbladel.github.io/aurelia-app-cli-sample/#/welcome) and type in the CLI: sum 1 3 and press enter. Press now key up and the same command is displayed.
Currently the history is kept until you type CLS. That will be improved in a later version of the plugin, were I will store the command history in the browser storage.
#### a CLI should never block the main application
WE run CLI commands as a promise. By doing so, they never block the main application.
#### a CLI should be able to run multiple commands in parallel
The sum commands takes deliberably 2 seconds, so try to run 5 or more and you'll see they run nicely in parallel.

### a CLI should be ready to cope with errors
Go to the [sample application](https://paulvanbladel.github.io/aurelia-app-cli-sample/#/welcome) and type in the CLI: EchoMe "hi there". 
We get the expected result. Type now: EchoMe xxx and you'll get an error, nicely colored in red.
#### bonus: aurelia-app-cli allows also graphical output
Normal CLI's running in the operating system can only provide text output. Aureli-app-cli allows also to return graphics.
Go to the [sample application](https://paulvanbladel.github.io/aurelia-app-cli-sample/#/welcome) and type in the CLI: GitHubUserInfo paulvanbladel and you'll meet the author of the plugin :)

## where are sources?
[aurelia-app-cli](https://github.com/paulvanbladel/aurelia-app-cli)
[aurelia-app-cli-sample](https://github.com/paulvanbladel/aurelia-app-cli-sample)
In an next post, I'll dive a bit deeper into the internals of this plugin.

Cheers

paul.