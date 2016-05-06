+++
title = "On Weave"
description = ""
date = "2016-04-07T23:39:40+08:00"
+++

I have been looking for a simple (as in [bus factor](https://en.wikipedia.org/wiki/Bus_factor) of 0) deployment solution for single-tenanted and microservice applications for a while now, which sort of rules out tonnes of sophisticated tools out there. I read this blog post a couple of days ago: [The fastest path to Docker on ECS: microservice deployment on Amazon EC2 Container Service with Weave Net](https://www.weave.works/guides/service-discovery-and-load-balancing-with-weave-on-amazon-ecs-2/).  I was really impressed with the simplicity of Weave, so I decided to port the setup to a Terraform module because Infrastructure-as-code is **it**,  and I plan to improve on it for a deployment solution I'm working on. You can find the Terraform module [in this github repo]( https://github.com/shuaibiyy/ecs-weave-terraform-microservice-demo). I think I've found a usable arsenal of deployment tools that work well together, and it includes Weave.  I plan on writing about my solution once it works out there.