+++
date = "2016-02-09T15:35:23+08:00"
description = ""
title = "Using Stormpath on a Java Backend"

+++

In a [previous post](http://shuaib.me/stormpath/), I talked about our motivation for picking Stormpath as the user management solution for [Distinction.ng](https://distinction.ng). To recap, our app is built using Java on the backend and React on the front-end. In this post, I'll go through how we got Stormpath working on our Java backend.

We are making use of the Stormpath Java Servlet Plugin, which has an excellent [getting started guide](https://docs.stormpath.com/java/servlet-plugin/). The plugin is mostly configuration driven and comes with reasonable defaults. Most of your application-specific configuration should reside in a file called `stormpath.properties`.

Our backend API is RESTful, so we started by figuring out the endpoints we wanted to authenticate, and then simply added lines like these to our `stormpath.properties` file:

    stormpath.web.uris./api/exams/** = authc
    stormpath.web.uris./api/subjects/** = authc
    stormpath.web.uris./api/questions/** = authc

By default, the Stormpath servlet plugin comes with a bunch of preconfigured endpoints, e.g. `/login, /register, /oauth/token` e.t.c. for different user management functions. We wanted our endpoints to be prefixed with `/api`, so we did some renaming in our `stormpath.properties` file:

    stormpath.web.logout.uri = /api/logout
    stormpath.web.register.uri = /api/register
    stormpath.web.accessToken.uri = /api/oauth/token

The property name for the HTTP basic authentication endpoint is `stormpath.web.login.uri` with a default value of `/login`.  We prefer to use token authentication.

Our backend API and front-end are located on separate domains, so we needed to specify origins that are allowed to request for access tokens by adding them to the properties file:

    stormpath.web.accessToken.origin.authorizer.originUris = http://localhost:3500, https://development-1.distinction.ng

The values above are for local development, and we update them based on environment, i.e. development, staging and production. One way of updating the URIs is programmatically via environment variables, which is also [12-Factor App](http://12factor.net/) compliant. Additionally, we needed to setup a CORS filter to handle cross-domain communication between the client and server. If you aren't familiar with CORS, MDN has an excellent [write-up](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS). For that, we created a Jersey filter like the following:

	import com.stormpath.sdk.servlet.filter.HttpFilter;
	// Imports skipped
    public class CORSStormpathFilter extends HttpFilter {
    
        @Override
        protected void filter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
                throws Exception {
            String origin = request.getHeader("Origin");
    
		    // Ensure the origin is allowed based on the environment
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

Next, we needed to ensure HTTP requests to Stormpath endpoints pass through `CORSStormpathFilter`, otherwise they wouldn't work on the browser because of CORS. Again, we added to the properties file:

	# Create URI filter called cors
	stormpath.web.filters.cors = com.flexisaf.cbt.filter.CORSStormpathFilter
	
	# Specify cors filter for desired endpoints
    stormpath.web.uris./api/logout = cors
    stormpath.web.uris./api/register = cors
    stormpath.web.uris./api/oauth/token = cors

With that, our backend API was ready to accept user registration, authenticate users, and perform other functions supported by the Stormpath Java Servlet Plugin.