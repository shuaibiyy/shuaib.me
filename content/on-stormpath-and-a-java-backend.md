+++
date = "2016-02-09T15:35:23+08:00"
description = ""
title = "On Stormpath and a Java Backend"

+++

In a [previous post](http://callme.ninja/stormpath/), I talked about our motivation for picking Stormpath as a user management solution for [Distinction.ng](https://distinction.ng). To recap, our app is built using Java on the backend and React on the front-end. In this post, I'll go through how we got Stormpath working in our Java stack.

We made use of Stormpath servlet plugin, which has an excellent [getting started guide](https://docs.stormpath.com/java/servlet-plugin/). The plugin is mostly configuration driven and comes with reasonable defaults. Most of your application-specific configuration should reside in a file called `stormpath.properties`.

Our backend is RESTful, so we started by figuring out the endpoints we wanted to authenticate, and then it was a matter of adding lines like these to `stormpath.properties` file:

    stormpath.web.uris./api/exams/** = authc
    stormpath.web.uris./api/subjects/** = authc
    stormpath.web.uris./api/questions/** = authc

By default, the Stormpath servlet plugin comes with a bunch of preconfigured endpoints, e.g. `/login, /register, /oauth/token` e.t.c. We wanted our endpoints to be prefixed with `/api`, so we did some renaming by adding to the properties file:

    stormpath.web.logout.uri = /api/logout
    stormpath.web.register.uri = /api/register
    stormpath.web.accessToken.uri = /api/oauth/token

We make use of token authentication as opposed to HTTP basic authentication, hence the `/api/oauth/token` endpoint.

Our backend API and front-end are located on separate domains, so we need to specify origins that are allowed to request for access tokens by adding to the properties file:

    stormpath.web.accessToken.origin.authorizer.originUris = http://localhost:3500

The value above is for local development, and we update it based on the current environment, e.g. development, staging. Additionally, we needed to setup a CORS filter to handle cross-domain communication between the client and server. If you aren't familiar with CORS, MDN has an excellent [write-up](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS). For that, we created a Jersey filter that looks like:

	import com.stormpath.sdk.servlet.filter.HttpFilter;
	// Imports skipped
    public class CORSStormpathFilter extends HttpFilter {
    
        @Override
        protected void filter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
                throws Exception {
            String origin = request.getHeader("Origin");
    
		    // Ensures the origin is allowed based on the environment
            checkOrigin(origin);
    
            // Handle CORS preflight
            if (request.getMethod().equalsIgnoreCase("OPTIONS")) {
                response.setHeader("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS, HEAD");
                response.setHeader("Access-Control-Allow-Headers",
                        "origin, content-type, accept, authorization, X-xsrf-token");
                response.setHeader("Access-Control-Max-Age", "1209600");
            }
            response.setHeader("Access-Control-Allow-Credentials", "true");
            response.setHeader("Access-Control-Allow-Origin", origin);
    
            super.filter(request, response, chain);
        }
    }

Next we needed to ensure requests go through the filter we created, so we again added to the properties files:

    stormpath.web.uris./api/logout = cors
    stormpath.web.uris./api/register = cors
    stormpath.web.uris./api/oauth/token = cors

With that, our backend was ready to accept user registration, authenticate users, and perform other functionality supported by the Stormpath plugin.