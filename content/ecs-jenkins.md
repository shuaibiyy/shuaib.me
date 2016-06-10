+++
title = "ECS-Powered Jenkins"
description = "Jenkins running in an ECS service with slaves running in ECS tasks"
date = "2016-05-13T03:33:00+08:00"
+++
 
I was recently in search of a scalable and cost-effective Jenkins setup using docker containers. I found what I believe satisfies that requirement by running Jenkins on [Amazon EC2 Container Service (ECS)](https://aws.amazon.com/ecs/). The simplest form of the setup involves running a single Jenkins master in an ECS cluster. To run builds in slaves, I use the [Amazon ECS Jenkins plugin](https://wiki.jenkinsci.org/display/JENKINS/Amazon+EC2+Container+Service+Plugin).

The way it works is, you add an ECS cluster as what is called a "Cloud" in the "Manage Jenkins" section of Jenkins configuration, and an "ECS slave template" that describes a docker image and its resource constraints. When you define your job configuration, you have to specify a restriction for where the job can run that matches a label you provided while declaring a slave template. The setup steps are well documented in the ECS plugin page.

I created a Terraform module that automates provisioning Jenkins on ECS using Terraform, it also provides a Terraform script for building and releasing your custom Jenkins image to [Amazon EC2 Container Registry (ECR)](https://aws.amazon.com/ecr/). You can find the Github repo [here](https://github.com/shuaibiyy/ecs-jenkins).

With everything set up - when builds are run, the ECS plugin starts an ECS task running a docker container from a configured slave template docker image and runs the build on it. ECS tasks are ephemeral, so once the build completes or fails, the task gets cleaned up. Here's an illustration of how everything ties up together:

![ECS Jenkins](https://rawgit.com/shuaibiyy/shuaib.me/master/themes/hugo-cactus-theme/images/ecs-jenkins.png)

In conclusion, I find that this is a superior approach over spot instances because builds are unlikely to be terminated abruptly and better than long running Jenkins slaves because of their upfront resource commitment. I am yet to see how this works with autoscaling though, as I imagine that if the provisioned EC2 instances don't have the capacity to serve the requested builds, new instances should be spawned up based on the autoscaling policy.