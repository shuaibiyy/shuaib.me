+++
title = "HAProxy Configuration Management with Cosmos and Cosmonaut"
description = "Enabling Networked and Containerised Deployments Using Dynamic HAProxy Configurations"
date = "2016-05-08T02:18:18+08:00"
+++

## Background
Deployment solutions for all sorts of application architectures are pretty common and well-known these days. A month ago, I set out to find tooling that could handle deploying a couple hundred single-tenanted applications in containers, and I found some, but nothing simple enough for the bus factor I had in mind. I wanted to leverage existing simple tools without buying into configuration heavy machinery. I finally settled on a set of tools that work well, but I had to build what I found to be missing pieces. The set of tools consists of the following:

* [Amazon ECS](https://aws.amazon.com/documentation/ecs/) for orchestration and scheduling.
* [Docker](https://www.docker.com/what-docker) mainly because of ECS.
* [HAProxy]() for load balancing and URL routing.
* [Weave NET](https://www.weave.works/products/weave-net/) to provide an overlay network for Docker; bridging together multiple hosts and allowing containers to find each other.
* [Terraform](https://www.terraform.io/) for provisioning the entire infrastructure.
* [Cosmos](https://github.com/shuaibiyy/cosmos) for managing HAProxy configurations.
* [Cosmonaut](https://github.com/shuaibiyy/cosmonaut) for reloading HAProxy when container lifecycle events occur.

## Terminologies
A couple of terms and their meanings as understood by Cosmos and Cosmonaut.

* Service: A service is a name given to containers that serve the same purpose. Containers are grouped as a service using an environment variable.
* A container is a docker container, and an instance of a service is a single container.
* HAProxy config is a HAProxy configuration file, typically found in a file named `haproxy.cfg`.

## Cosmos
Cosmos is a tool for managing and generating HAProxy configurations for hosts running services in containers behind a HAProxy. Cosmos receives a payload describing the state of services and returns a HAProxy config that matches that state. It also stores the data of past services, so their configurations persist across future HAProxy configs as long as there are running instances, i.e. containers, of them.

## Cosmonaut
Cosmonaut is a tool for monitoring docker hosts and reloading their HAProxy configurations. When a docker event relevant to Cosmonaut occurs, it gathers information about the event and current state of services running on the host, and sends that information in a request to Cosmos, which then returns a HAProxy config. Cosmonaut finally reloads the host's HAProxy with the config it received. Currently, Cosmonaut expects HAProxy to be running in a [container](https://github.com/rstiller/dockerfiles/tree/master/haproxy), and it updates it via a docker exec command.

