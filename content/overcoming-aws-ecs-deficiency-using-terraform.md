+++
title = "Overcoming an AWS ECS Deficiency Using Terraform"
description = ""
date = "2016-04-27T05:58:17+08:00"
+++

While trying to deploy an application using ECS, I hit into a popular deficiency, which is the inability to securely and dynamically supply values to environment variables defined in ECS task definitions. Docker serves that need by providing an --env-file option for its run command. For ECS however, there are outstanding feature requests that have been raised on Github [here](https://github.com/aws/amazon-ecs-agent/issues/3) and [here](https://github.com/aws/amazon-ecs-agent/issues/247). The ECS scheduler being a closed-source system means there's no community to pick up backlog items that AWS engineers may never get around to.

In this particular case, I was lucky enough to already be using [Terraform](https://www.terraform.io/) to provision my infrastructure, which made for an easy solution. For those unfamiliar with Terraform, it's a tool from HashiCorp, the creators of Vagrant, that lets you write configurations in a language called HCL, which you can then run to provision resources on any supported cloud provider. Terraform codifies infrastructure, enabling what is known as Infrastructure as Code.

Without further ado, I have a task definition called jenkins.json that initially contained this segment:

	{
	  "name": "jenkins-backup",
	  "image": "istepanov/backup-to-s3",
      "memory": 128,
	  "cpu": 10,
      "essential": false,
      "environment": [
        {
          "name": "ACCESS_KEY",
          "value": "very_bad_practice"
        },
        {
          "name": "SECRET_KEY",
          "value": "dont_do_this"
        },
        {
          "name": "S3_PATH",
          "value": "s3://allhardcode/d/"
        },
        {
          "name": "CRON_SCHEDULE",
          "value": "0 12 * * *"
        }
      ],
      "mountPoints": [
        {
          "sourceVolume": "jenkins-home",
          "containerPath": "/data"
        }
      ]
    }
  
Clearly, the issue with the task definition above is that it is exposing credentials, and forcing you to hardcode values that you might want to set dynamically. The solution is to use [__Terraform's template_file__](https://www.terraform.io/docs/providers/template/r/file.html). We can do that by creating a file called jenkins.json.tpl (the .tpl extension is not required), and it'll contain this:

	{
	  "name": "jenkins-backup",
	  "image": "istepanov/backup-to-s3",
      "memory": 128,
	  "cpu": 10,
      "essential": false,
      "environment": [
        {
          "name": "ACCESS_KEY",
          "value": "${aws_access_key}"
        },
        {
          "name": "SECRET_KEY",
          "value": "${aws_secret_key}"
        },
        {
          "name": "S3_PATH",
          "value": "s3://${s3_bucket}/${s3_path}/"
        },
        {
          "name": "CRON_SCHEDULE",
          "value": "0 12 * * *"
        }
      ],
      "mountPoints": [
        {
          "sourceVolume": "jenkins-home",
          "containerPath": "/data"
        }
      ]
    }
In our Terraform scripts, we declare the template file as a resource:

	resource "template_file" "jenkins_task_template" {
	  template = "${file("jenkins.json.tpl")}"
	
	  vars {
	    aws_access_key = "${var.aws_access_key}"
	    aws_secret_key = "${var.aws_secret_key}"
	    s3_bucket = "${var.s3_bucket}"
	    s3_path = "${var.s3_path}"
	  }
	}
The variable names in the vars section of the template_file definition match the placeholders in jenkins.json.tpl, and their values will be interpolated in the rendered template when it is used by a service like this one:

	resource "aws_ecs_task_definition" "jenkins" {
	  family = "jenkins"
	  container_definitions = "${template_file.jenkins_task_template.rendered}"
	
	  volume {
	    name = "jenkins-home"
	    host_path = "/ecs/jenkins-home"
	  }
	}
Terraform provides all the means for [specifying variables](https://www.terraform.io/docs/configuration/variables.html) that one could ask for, and you can take advantage of them in your ECS task definitions and potentially lots of static configuration files out there. Sounds like a pretty sweet deal to me.