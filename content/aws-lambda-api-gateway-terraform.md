+++
title = "On AWS Lambda, API Gateway and Terraform"
description = ""
date = "2016-04-18T12:50:49+08:00"
+++

[AWS Lambda](https://aws.amazon.com/lambda/) is arguably the most exciting service released in AWS since EC2. Lambda is a service that lets you run code on someone else's machine, in this case EC2. All you need to do is pick the runtime your code can run in, and provide the code. Currently, the supported runtimes are:

* Node.js: v0.10.36, v4.3.2
* Java: Java 8
* Python: Python 2.7

Developing applications using Lambda differs from the way we are typically used to, in terms of codebase management, tooling, frameworks, testing and deployment. On one hand, Lambda offers us the entire AWS ecosystem with simple configurations, and on the other, it requires us to rethink how we approach building even small applications. There aren't yet enough success stories and best practices out there to give one the confidence to build large applications using Lambda, but there's enough information to start farming out computation heavy processes to Lambda. Lambda especially shines because of its ability to scale along with its workload.

[API Gateway](https://aws.amazon.com/api-gateway/) is another exciting service on AWS that aims to ease the task of creating APIs. You define your resources and their models, request transformations, locations where requests should be proxied to, response transformations; and you get a functioning API without deploying a single machine. An API Gateway endpoint can use a Lambda function as its backend, which is the sweet spot touted by serverless architecture advocates.

I recently created a small project using Lambda and API Gateway. When deployed, the application provides an API endpoint that can be used to generate a `haproxy.cfg` file based on parameters provided. You can find the project source [here](https://github.com/shuaibiyy/haproxy-config-generator).

One major pain point of using Lambda and API Gateway is the difficulty of setting things up, so the project uses [Terraform](https://terraform.io) to ease that difficulty. Terraform is a tool that lets you define configurations, which it can run to provision resources on datacenters by providers such as AWS, Azure and Google Cloud. In this project, Terraform is used to provision the Lambda function and API Gateway resources. With Terraform installed, the project can be deployed by simply invoking:
```bash
terraform apply
```
and torn down using:
```bash
terraform destroy
```
Like every system in its early life, API Gateway and Lambda have minor bugs and areas of improvement. One particular bug I couldn't find a sensible workaround for is API Gateway failing to have the right permissions to talk with Lambda after deployment. The solution is to perform a ceremony described in [this youtube video](https://www.youtube.com/watch?v=H4LM_jw5zzs).

Overall, the combination of these technologies is lethal, and I'm interested in seeing how functionality in existing applications can be chipped away to harness the strengths of these so-called serverless architectures.