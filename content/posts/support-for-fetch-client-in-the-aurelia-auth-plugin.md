+++
date = "2015-10-29T09:03:37+00:00"
draft = false
title = "Support for Fetch client in the aurelia-auth plugin"
slug = "support-for-fetch-client-in-the-aurelia-auth-plugin"

+++

## Introduction
In aurelia, there are currently two options for http backend communication: aurelia-fetch-client and aurelia-http-client. The preferred option is to use the Fetch client since is based on a real standard.

In the aurelia-auth plugin, some kind of http configuration needs to be done, namely the injection of the JWT token in each 'authenticated' request. By doing so, you don't need to worry about this when sending a particular http request to your backend, the aurelia-auth plugin will automatically take the JWT token from the browser storage and add it to the http request message.

## How configuring aurelia-fetch-client?
In your aurelia app file, inject the {FetchConfig} class from aurelia-auth. We need to explicitely opt-in for the configuration of your fetch client by calling the configure function of the FetchConfig class:
```language-javascript
import 'bootstrap';
import {inject} from 'aurelia-framework';
import {Router} from 'aurelia-router';
import {FetchConfig} from 'aurelia-auth';
@inject(Router,FetchConfig, AppRouterConfig )
export class App {

  constructor(router, fetchConfig, appRouterConfig){
    this.router = router;
    this.fetchConfig = fetchConfig;
  }

  activate(){
    this.fetchConfig.configure();
  }
}
```
Note that we are doing the import of FetchConfig slightly different than how it was done for aurelia-http-client, where you needed to specify the interal file name of 
```language-javascript
import HttpClientConfig from 'aurelia-auth/app.httpClient.config';
```
HttpClientConfig was exposed as a default export in aurelia-auth. That's why there was no need for the curly braces when doing the import, but you need to specify the name of the class file when doing the import and that's a bit bizarre since the index file of the plugin is there the keep internals of the plugin private. Anyhow I kept this way of working for aurelia-http-client but used the 'better' approach for aurelia-fetch-client.

## Using the fetch client
The usage of the Fetch client is slightly different:
```language-javascript
import {inject} from 'aurelia-framework';
import {HttpClient} from 'aurelia-fetch-client';

@inject(HttpClient)
export class CustomerFetch{
  heading = 'Customer management';
  customers = [];

  url = 'api/customer';

  constructor(http){
    this.http = http;
  }

  activate(){

   return this.http.fetch(this.url)
   .then(response =>  response.json())
   .then(c => this.customers = c);
}

}

```
## sources
The plugin can be found on github:
[https://github.com/paulvanbladel/aurelia-auth](https://github.com/paulvanbladel/aurelia-auth)

I try to maintain also a (nodeJs) full stack sample where the plugin is consumed:
[https://github.com/paulvanbladel/aurelia-auth-sample/](https://github.com/paulvanbladel/aurelia-auth-sample/)

## Conclusion
For new projects, definitely use the aurelia-fetch-client. Especially since it's now supported by aurelia-auth :)
Cheers.
