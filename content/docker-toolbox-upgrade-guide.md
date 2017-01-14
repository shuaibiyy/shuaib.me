+++
title = "Docker Toolbox Upgrade Guide"
description = ""
date = "2016-04-25T17:55:00+08:00"
+++

1. Get the latest docker toolbox installer from the [docker website](https://www.docker.com/products/docker-toolbox).
1. Stop all running docker machines, e.g.

		$ docker-machine stop dev
1.  Run the toolbox installer and agree to any upgrade request.
1. Once the installation is complete, you'll need to upgrade the docker machine, e.g.

		$ docker-machine upgrade dev
1. Start the docker machine e.g.

		$ docker-machine start dev
1. If the docker machine's IP changes, you'll need to regenerate its certs, e.g.

		$ docker-machine regenerate-certs dev
1. Configure your shell to use the docker machine as your docker engine, e.g.

		$ eval "$(docker-machine env dev)"

That's it.
