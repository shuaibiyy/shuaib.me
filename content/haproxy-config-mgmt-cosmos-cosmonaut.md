+++
title = "HAProxy Configuration Management with Cosmos and Cosmonaut"
description = "Enabling Networked and Containerised Deployments Using Dynamic HAProxy Configurations"
date = "2016-05-08T02:18:18+08:00"
+++

## Background
Deployment solutions for all sorts of application architectures are pretty common and well-known these days. A month ago, I set out to find tooling that could handle deploying a couple hundred single-tenanted applications in containers, and I found some, but nothing simple enough for the bus factor I had in mind. I wanted to leverage existing simple tools without buying into configuration heavy machinery. I finally settled on a set of tools that work well, but I had to build what I found to be missing pieces. The set of tools consists of the following:

* [Amazon ECS](https://aws.amazon.com/documentation/ecs/) for orchestration and scheduling.
* [Docker](https://www.docker.com/what-docker) mainly because of ECS.
* [HAProxy](http://www.haproxy.org/) for load balancing and URL routing.
* [Weave NET](https://www.weave.works/products/weave-net/) to provide an overlay network for Docker; bridging together multiple hosts and allowing containers to find each other.
* [Terraform](https://www.terraform.io/) for provisioning the entire infrastructure.
* [Cosmos](https://github.com/shuaibiyy/cosmos) for managing HAProxy configurations.
* [Cosmonaut](https://github.com/shuaibiyy/cosmonaut) for reloading HAProxy when container lifecycle events occur.

****************************************

## Terminologies
A couple of terms and their meanings as understood by Cosmos and Cosmonaut.

* Service: A service is a name given to containers that serve the same purpose. Containers are grouped as a service using an environment variable.
* A container is a docker container, and an instance of a service is a single container.
* HAProxy config is a HAProxy configuration file, typically found in a file named `haproxy.cfg`.

****************************************

## [Cosmos](https://github.com/shuaibiyy/cosmos)
Cosmos is a tool for managing and generating HAProxy configurations for hosts running services in containers behind a HAProxy. Cosmos receives a payload describing the state of services and returns a HAProxy config that matches that state. It also stores the data of past services, so their configurations persist across future HAProxy configs as long as there are running instances, i.e. containers, of them.

**Sample request to Cosmonaut:**

	{
	  "tableName": "astro",
	  "runningServices": [{
	      "serviceName": "app1",
	      "id": "a23nj53h3j4",
	      "ip": "192.168.1.9:80"
	    },
	    {
	      "serviceName": "app1",
	      "id": "jk3243j54jl",
	      "ip": "192.168.1.8:80"
	    }
	  ],
	  "candidateServices": [{
	      "serviceName": "app1",
	      "configMode": "host",
	      "predicate": "first.example.com",
	      "cookie": "JSESSIONID",
	      "containers": [{
	          "id": "a23nj53h3j4",
	          "ip": "192.168.1.9:80"
	        }
	      ]
	    },
	    {
	      "serviceName": "app2",
	      "configMode": "host",
	      "predicate": "second.example.com",
	      "cookie": "JSESSIONID",
	      "containers": [{
	          "id": "das843j3h3k",
	          "ip": "192.168.1.10:80"
	        },
	        {
	          "id": "fds32k4354f",
	          "ip": "192.168.1.11:80"
	        }
	      ]
	    }
	  ]
	}

**Explanation:**

* **tableName**: name of DynamoDB table where configurations will be stored.
* **runningServices**: instances of services running within the weave network.
* **candidateServices**:  instances that are new to the weave network and do not yet exist in the HAProxy config.
* **configMode**: type of routing. It can be either `Path` or `Host`. In `Path` mode, the URL path is used to determine which backend to forward the request to. In `Host` mode, the HTTP host header is used to determine which backend to forward the request to.
	*Defaults to `host` mode.*
* **serviceName**: name of service the containers belong to.
* **predicate**: value used along with mode to determine which service a request will be forwarded to. 
In `Path` mode, the predicate looks like: 
	
			acl <cluster> url_beg /<predicate>
		
	In `Host` mode:
	
			acl <cluster> hdr(host) -i <predicate>
		
* **cookie**: name of cookie to be used for sticky sessions. If not defined, sticky sessions will not be configured.
* **containers**: key-value pairs of container ids and their corresponding IP addresses.

**HAProxy config generated from request:**

	global
	   log 127.0.0.1   local0
	   log 127.0.0.1   local1 notice
	   maxconn 4096
	   user haproxy
	   group haproxy
	   daemon
	
	defaults
	    log global
	    mode http
	    option httplog
	    option dontlognull
	    option forwardfor
	    option http-server-close
	    timeout connect 5000
	    timeout client 50000
	    timeout server 50000
	
	# Define frontends
	
	frontend http
	    bind :80
	    
	    acl app1 hdr(host) -i first.example.com
	    use_backend app1 if app1
	    
	    acl app2 hdr(host) -i second.example.com
	    use_backend app2 if app2
	    
	
	# Define backends
	
	backend app1
	    mode http
	    balance roundrobin
	    option forwardfor
	    cookie JSESSIONID prefix nocache
	    
	    server a23nj53h3j4 192.168.1.9:80 check cookie JSESSIONID
	    
	backend app2
	    mode http
	    balance roundrobin
	    option forwardfor
	    cookie JSESSIONID prefix nocache
	    
	    server das843j3h3k 192.168.1.10:80 check cookie JSESSIONID
	    
	    server fds32k4354f 192.168.1.11:80 check cookie JSESSIONID


## [Cosmonaut](https://github.com/shuaibiyy/cosmonaut)
Cosmonaut is a tool for monitoring docker hosts and reloading their HAProxy configurations. When a docker event relevant to Cosmonaut occurs, it gathers information about the event and current state of services running on the host, and sends that information in a request to Cosmos, which then returns a HAProxy config. Cosmonaut finally reloads the host's HAProxy with the config it received. Currently, Cosmonaut expects HAProxy to be running in a [container](https://github.com/rstiller/dockerfiles/tree/master/haproxy), and it updates it via a docker exec command.

