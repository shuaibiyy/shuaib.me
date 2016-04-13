+++
title = "On Stormpath and SSL Termination"
description = ""
date = "2016-04-13T14:57:45+08:00"
+++

Our application sits behind Amazon's Elastic Load Balancer (ELB), which is a convenient place for terminating SSL connections. We use the Stormpath servlet plugin and its token authentication feature for our Stormpath integration. For some reason though, requests from users trying to authenticate with our application errored out with the following message:
```bash
[http-listener-1(3)] DEBUG c.s.s.s.mvc.AccessTokenController - OAuth Access Token request failed.
com.stormpath.sdk.servlet.filter.oauth.OauthException: A secure HTTPS connection is required for token requests - this is a requirement of the OAuth 2 specification.
```
A look at the request headers from clients showed that `X-Forwarded-Proto` headers were getting set. The `X-Forwarded-Proto` header is responsible for informing the server of the protocol at the origin where a request was initiated. In our case, we wanted Glassfish to treat connections that had `X-Forwarded-Proto` header values of `https` as secured, so we configured it to do so. That however, still didn't solve the problem.

After getting in touch with the excellent support folks at Stormpath, they stated that we were facing the issue described in this [Github issue](https://github.com/stormpath/stormpath-sdk-java/issues/139).

They also offered the following workaround:

1. Create this Factory:
	```java
	public class SecureResolverFactory extends ConfigSingletonFactory<Resolver<Boolean>> {
	
	    public static final String LOCALHOST_RESOLVER = "stormpath.web.localhost.resolver";
	
	    @Override
	    protected Resolver<Boolean> createInstance(ServletContext servletContext) throws Exception {
	        return new SecureForwardedProtoAwareResolver(new IsHTTPSForwardedProtoResolver(), new SecureRequiredExceptForLocalhostResolver(getConfig().getInstance(LOCALHOST_RESOLVER)));
	    }
	}
	```

2. In stormpath.properties add:
	```bash
	stormpath.web.accessToken.authorizer.secure.resolver = com.mycompany.resolver.SecureResolverFactory
	```

3. Create these Resolvers:
	```java
	public class IsHTTPSForwardedProtoResolver implements Resolver<Boolean> {
	
	    private static final String HEADER_FORWARDED_PROTO = "X-Forwarded-Proto";
	
	    @Override
	    public Boolean get(HttpServletRequest request, HttpServletResponse response) {
	        String protocol = request.getHeader(HEADER_FORWARDED_PROTO);
	        if (protocol != null && protocol.equalsIgnoreCase("https")) {
	            return true;
	        }
	        return false;
	    }
	}
	
	public class SecureForwardedProtoAwareResolver implements Resolver<Boolean> {
	
	    private final Resolver<Boolean> isHTTPSForwardedProtoResolver;
	    private final Resolver<Boolean> secureRequiredExceptForLocalhostResolver;
	
	    public SecureForwardedProtoAwareResolver(Resolver<Boolean> isHTTPSForwardedProtoResolver, Resolver<Boolean> secureRequiredExceptForLocalhostResolver) {
	        Assert.notNull(isHTTPSForwardedProtoResolver, "isHTTPSForwardedProtoResolver resolver cannot be null.");
	        Assert.notNull(secureRequiredExceptForLocalhostResolver, "secureRequiredExceptForLocalhost resolver cannot be null.");
	        this.isHTTPSForwardedProtoResolver = isHTTPSForwardedProtoResolver;
	        this.secureRequiredExceptForLocalhostResolver = secureRequiredExceptForLocalhostResolver;
	    }
	
	    @Override
	    public Boolean get(HttpServletRequest request, HttpServletResponse response) {
	        if (this.isHTTPSForwardedProtoResolver.get(request, response)) {
	            return false;
	        }
	        return this.secureRequiredExceptForLocalhostResolver.get(request, response);
	    }
	}
	```
That's it, Stormpath now considers requests with `X-Forwarded-Proto`  values of `https` as secured.