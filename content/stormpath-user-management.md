+++
date = "2016-02-06T01:31:31+08:00"
description = ""
title = "On Stormpath for User Management"

+++

After the beta release of [Distinction.ng](http://distinction.ng), allowing users to register an account that they can use to track their progress was one of our next priorities. User management being an after-thought, coupled with a hard deadline looming over us, we set out to get something out fast. I had heard about [Stormpath](https://stormpath.com/) from an episode of [JavaScript Jabber](https://devchat.tv/js-jabber/) where a Stormpath engineer was the main guest. I had also checked out Auth0 after seeing their excellent blog post on implementing authentication in [React](https://auth0.com/blog/2015/04/09/adding-authentication-to-your-react-flux-app/) and [code samples](https://github.com/auth0) on authenticating Single Page Applications (SPA). After giving some thought to both the pricing plans for both Stormpath and Auth0, we decided to go with Stormpath.

What is Stormpath? Stormpath is a platform that takes care of your application's user management layer, providing what amounts to Authentication and Authorisation as a Service.

Stormpath has built an impressive toolset for numerous platforms, and they have even more in the pipeline. Distinction's stack consists of a Java EE backend and a ReactJS front-end. For web applications written in Java, one of the tools available is the [Stormpath Servlet Plugin](https://docs.stormpath.com/java/servlet-plugin/). The servlet makes it super easy to add registration, login, authentication and authorisation functionalities to your application. For front-end applications written in React, Stormpath provides the [Stormpath React SDK](https://github.com/stormpath/stormpath-sdk-react), a fairly new project that tries to do a lot of the work required when adding authentication to your app. In our project, however, we opted not to use the SDK. The SDK's code was fairly small and simple, so we decided it would be more productive in the long run to implement a narrower set of functionalities tailored to our use-case and consistent with our application's design.

In the next posts to come, I'll go through setting up Stormpath on the backend and front-end, and conclude with lessons learnt.