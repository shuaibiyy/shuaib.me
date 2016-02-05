+++
date = "2016-02-06T01:31:31+08:00"
description = ""
title = "Stormpath"

+++

We had the need to setup user management in an [project](http://distinction.ng) I was working on recently. User management being an after-thought, coupled with a hard deadline looming over us, we set out to bang something out in very limited time. I had heard about Stormpath from an episode of JavaScript Jabber where they had a Stormpath engineer as the main guest. I had also checked out Auth0 after seeing their excellent blog post on implementing authentication in [React](https://auth0.com/blog/2015/04/09/adding-authentication-to-your-react-flux-app/) and [code samples](https://github.com/auth0) on authenticating Single Page Applications (SPA). After giving some thought to both the pricing options for Stormpath and Auth0, we decided to go with Stormpath.

What is Stormpath? Stormpath is a platform that takes care of your application's user management layer, providing what amounts to Authentication and Authorisation as a Service. Our stack consists of a Java EE backend and a ReactJS front-end. In separate posts, I go through setting up Stormpath on the backend and front-end.

Stormpath has built an impressive toolset for numerous platforms, and they appear to have more in the pipeline. For web applications written in Java, one of the tools available is the [Stormpath Servlet Plugin](https://docs.stormpath.com/java/servlet-plugin/). The servlet makes it super easy to add registration, login, authentication and authorisation functionalities to your application. For front-end applications, Stormpath provides the [Stormpath React SDK](https://github.com/stormpath/stormpath-sdk-react), a fairly new project that tries to do a lot of the work required when adding authentication to your app. In our project, however, we opted not to use the SDK. The SDK's code was fairly small and simple, so we decided it would be more productive in the long run to implement a narrower set of functionalities tailored to our use-case and consistent with our app's design.