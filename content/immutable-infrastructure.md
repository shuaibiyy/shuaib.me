+++
title = "On Immutable Infrastructure"
description = ""
date = "2016-05-06T19:44:10+08:00"
+++

I believe there's little convincing to be done on the merits of immutable deployment. With immutable deployment, the unit of abstraction is a machine image; a docker image, Amazon Machine Image (AMI), e.t.c. The idea is that your server, spawned from a machine image goes from being this special snowflake to [insert generic thing]. As opposed to meticulously maintaining a server and keeping track of its mutable configuration over time, you are able to provision it with a known configuration in a non-special way. Containerisation makes this easier because of the lightweight and wieldy characteristics of containers.

Last week, I listened to a talk on **immutable infrastructure**, titled [Immutable Infrastructure: Rise of the Machine Images](https://www.infoq.com/presentations/immutable-infrastructure). The speaker expands on immutable deployment by talking about  how certain ancillary aspects of applications are affected by immutability, such as logging, sessions, configuration and service discovery. I highly recommend that talk as the speaker puts into words vaguely defined practices that have emerged from this growing trend.