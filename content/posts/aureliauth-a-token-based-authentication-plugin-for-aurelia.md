+++
date = "2015-07-06T07:28:23+00:00"
draft = false
title = "Aureliauth, a token based authentication plugin for Aurelia"
slug = "aureliauth-a-token-based-authentication-plugin-for-aurelia"

+++

## What is Aureliauth
Aureliauth is a token-based authentication plugin for [Aurelia](http://aurelia.io/) with support for popular social authentication providers (Google, Twitter, Facebook, LinkedIn, Windows Live, FourSquare, Yahoo, Github ) and a local stragegy, i.e. simple username (email) and password.

Aureliauth is a port of the great [Sattelizer](https://github.com/sahat/satellizer/) library to ES6 and packaged as an Aurelia plugin.

Other OAuth1 and Oauth2 than the above mentioned providers can be simply added by editing the extensible configuration file.

Basically, Aureliauth does not use any cookies but relies on a JWT (Json web token) stored in the local storage of the browser:
![JWT in local storage](/content/images/2015/07/TokenViaDevelopmentTools.png)

Both **local storage** as well as **session storage** can be used (via the Aureliauth security configuration file).

The aurelia token will be sent automatically to your API when the user is authenticated.

![](/content/images/2015/07/authHeader.png)



## Where can I find more details and a sample app?
The sources are on my [github](https://github.com/paulvanbladel/aureliauth) space.

There is also a sample app: [aureliauth.opinionatedapps.com](http://aureliauth.opinionatedapps.com)

The sources of the sample app are also available on [github](https://github.com/paulvanbladel/aureliauth-sample)

The README contains a lot more information.
Enjoy !


