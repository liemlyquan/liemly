+++ 
draft = false
date = 2018-11-24T07:00:00+07:00
title = "Docker for local development"
description = ""
slug = ""
authors = []
tags = ["dev-tool"]
categories = []
externalLink = ""
series = []
+++

I first came across Docker two years ago, when I started working at Chopp. At that time, we are using Docker on our staging and production. Jenkins will run some tests through the code, building the Docker images, and deploy them to desired destinations. Now, after I joining Parcel Perform, I have had a chance to use Docker more extensively, even in local environment, and there are some noticeable changes compared to the old method that I could realize over a course of a month

## Simpler setup

With traditional method, when I am first on-boarded to a project, I will read through the documentation and install some required software and dependency. However, with Docker, I can simply run a docker-compose command to have everything up and running (err, I just realized we need to install Docker as a requirement). The same case applied for when I am starting a new development session. Of course, one can argue that this setup would be possible even with normal shell script. However, I imagine it would be harder when combining with the next 2 element

## Isolation
We have services which have their own dependent libraries, some of which require specific version. Thus, a requirement on one project could possibly break another. By having a container for each service, we can isolate the services environment, while still allowing them to communicate with each other.

### Compatibility
We have people using different OS, with different versions, and we have see no problems with setup on any computer. This would save quite some time for on-boarding, I believe. One more thing is that this would also ensure that our code would be the same across environments. There was a time at Chopp where I run into a problem only on staging and production. After several of hours tracing back, I figured out the problem was with the missing of a library the Docker images. If I used Docker on local environment, (I guess) this bug would have not had happened at the first place

## Unknown bugs
Despite its advantages, There are some weird bugs that I could not know why (yet) when using Docker at local development. Some of the supposed-to-be stable infrastructure services (databases, message queue) went wrong after a restart. I am quite sure that all of these are shut down gracefully, with no weird action, and I was not the only one in the team encountering these bugs. After that, there are some methods that I could do to make it up and running again, with ascending effectiveness
- Restart the container (less likely works though)
- Run docker-compose up force-recreate
- Remove the container, and run docker-compose up (actually, I wrote a function for this, since it is much effective than the other 2)