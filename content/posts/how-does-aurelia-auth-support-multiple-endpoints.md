+++
date = "2016-03-25T06:40:02+00:00"
draft = false
title = "How does aurelia-auth support multiple endpoints?"
slug = "how-does-aurelia-auth-support-multiple-endpoints"

+++

##Introduction
I sometimes get the question if aurelia-auth does support multiple endpoints. Sometimes more degrees of freedom are required when it comes to http configuration.
My solution for this, is 'close to the metal' but promises 100% degrees of freedom.

Let's demistify things. Watch carefully !

##How does aurelia-auth augment the Http Client?

Well, it's import to have a clear understanding how exactly the default Http Client is augemented by aurelia-auth and that we have the full freedom to use your own custom logic as well. Let's first show how aurelia-auth augments the Http Client by looking at app.fetch-httpClient.config.js :

```language-javascript
import {inject} from 'aurelia-dependency-injection';
import {HttpClient} from 'aurelia-fetch-client';
import 'isomorphic-fetch';
import {Authentication} from './authentication';

@inject(HttpClient, Authentication )
export class FetchConfig {
    constructor(httpClient, authService) {
        this.httpClient = httpClient;
        this.auth = authService;
    }

    configure() {
        this.httpClient.configure(httpConfig => {
            httpConfig
                .withDefaults({
                    headers: {
                        'Accept': 'application/json'
                    }
                })
                .withInterceptor(this.auth.tokenInterceptor);
        });
    }
}
```
So, basically a default Accept header is added and a request interceptor is injected. 'tokenInterceptor' can be found in authentication.js and is responsible for adding the bearer token to each request message making use of the default Http Client.

```language-javascript
get tokenInterceptor() {
        let config = this.config;
        let storage = this.storage;
        let auth = this;
        return {
            request(request) {
                if (auth.isAuthenticated() && config.httpInterceptor) {
                    let tokenName = config.tokenPrefix ?`${config.tokenPrefix}_${config.tokenName}` : config.tokenName;
                    let token = storage.get(tokenName);

                    if (config.authHeader && config.authToken) {
                        token = `${config.authToken} ${token}`;
                    }

                    request.headers.append(config.authHeader, token);
                }
                return request;
            }
        }
    }
```

So, this behavior is executed automatically when the HttpClient is configured as explained in the 'Configure the Fetch Client' of the repository's readme.

Now, what if we don't want this to happen for all Http request we make or imagine we have multiple end points and some of them do not require authentication.

The solution is very simple and based on standard ES 2016 OO techniques: simply derive a custom Http Client.

In aurelia-auth-sample we can find following custom Http Client:

```language-javascript
import {HttpClient} from 'aurelia-fetch-client';
import {inject} from 'aurelia-framework';
import 'isomorphic-fetch';
import {AuthService} from 'aurelia-auth';

@inject(AuthService)
export class CustomHttpClient extends HttpClient {
    constructor(auth) {
        super();
        this.configure(config => {
            config
                .withBaseUrl('http://localhost:4000/')
                .withDefaults({
                    credentials: 'same-origin',
                    headers: {
                        'Accept': 'application/json',
                        'X-Requested-With': 'Fetch'
                    }
                })
                //we call ourselves the interceptor which comes with aurelia-auth
                //obviously when this custom Http Client is used for services 
                //which don't need a bearer token, you should not inject the token interceptor
                .withInterceptor(auth.token_interceptor)
                //still we can augment the custom HttpClient with own interceptors
                .withInterceptor({
                    request(request) {
                        console.log(`Requesting ${request.method} ${request.url}`);
                        return request; // you can return a modified Request, or you can short-circuit the request by returning a Response
                    },
                    response(response) {
                        console.log(`Received ${response.status} ${response.url}`);
                        return response; // you can return a modified Response
                    }
                });
                });
    }
}
```
We use a simple ES 2015 OO technique: inheritance via the extends keyword. So, the above custom Http Client uses its own headers and baseUrl and has its own interception logic. Note, that you can still use the aurelia-auth standard interceptor for adding the bearer token.

As you can see, no specialised aurelia plugins needed for supporting multiple endpoints !

We can consume this custom Http Client as follows:

```language-javascript
import {inject, useView} from 'aurelia-framework';
import {CustomHttpClient} from './customHttpClient';
import 'isomorphic-fetch';
@inject(CustomHttpClient)
@useView('./customer.html')
export class Customer2{
  heading = 'Customer management with custom http service';
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

## Conclusion
The way we provide support for multiple endpoints is indeed 'close to the metal', but for something like http support, that's what you need.
