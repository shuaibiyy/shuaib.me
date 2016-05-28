+++
title = "Passing Key-value Program Arguments to Go Programs in Code"
description = "Trouble Passing Arguments to Terraform's Remote Config Command"
date = "2016-05-25T21:16:07+08:00"
+++

I'm currently working on [Topo](https://github.com/shuaibiyy/topo), a tool that aids provisioning multiple [Terraform](https://terraform.io) configurations of the same project. An example use-case where Topo might be a good solution is: say you want to provision multiple Jenkins servers for different teams, and you want to maintain the state of the resources so you can run Terraform to reapply as the configuration or capacity changes, or destroy.

I'm writing Topo in [Go](https://golang.org/), and I'm using the [go-sh](https://github.com/codeskyblue/go-sh) package to programmatically run shell processes. All was going well till I had to write the [Terraform remote config command](https://www.terraform.io/docs/commands/remote-config.html) for configuring remote state storage in S3. The command looks like this:
```bash
terraform remote config \
    -backend=s3 \
    -backend-config="bucket=terraform-state-prod" \
    -backend-config="key=network/terraform.tfstate" \
    -backend-config="region=us-east-1"
```
In my code, I wrote the go-sh command like this:
```go
sh.Command("terraform", "remote", "config",
	"-backend=s3", "-backend-config='bucket=jenkins-bucket'",
	"-backend-config='key=jenkins/terraform.tfstate'",
	sh.Dir("projects/jenkins")).Run()
```
That doesn't work though, and results in this error:
```
missing 'bucket' configuration

If the error message above mentions requiring or modifying configuration
options, these are set using the `-backend-config` flag. Example:
-backend-config="name=foo" to set the `name` configuration
```
The Terraform command works perfectly when run the command line. I spent a couple of hours trying stuff out. Terraform is a [Go](https://golang.org/) program, and it turns out that Go programs accept flags as key-value pairs(except for booleans) separated by equals sign ("="). Breaking up the arguments like below from "key='value'" to "key", "value" solves the problem:
```go
sh.Command("terraform", "remote", "config", "-backend=s3",
	"-backend-config", "bucket=jenkins-bucket",
	"-backend-config", "key=jenkins/terraform.tfstate",
	sh.Dir("projects/jenkins")).Run()
```