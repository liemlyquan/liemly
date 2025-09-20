+++ 
draft = false
date = 2020-08-02T07:00:00+07:00
title = "Initial experience with Kafka"
description = ""
slug = ""
authors = []
tags = ["message-queue"]
categories = []
externalLink = ""
series = []
+++

Before starting my job at Parcel Perform, the usage of Kafka, or message queue in general, would sound ridiculous to me. I thought to myself: "why do we have to add complexity to our system, instead of processing the data, we have to put it into a place, and then later on consume from that". However, after months of using that, I have started to realize how (mostly) wrong I am.
## Asynchronous processing
One thing to keep in mind is that here is that Kafka should only be used for asynchronous processing. One example is when a client sends a request to your system, they only need to know that the request went through, and do not need to know/see the result of the request immediately.
There could also be a mix of them.
For instance, when a user changes his account password, the user expects to see the change immediately, and receive a confirmation email afterward. Then, in this case, we should be able to combine both. We can return the password change result synchronously, and sending email to user asynchronously

## Feature
### Log retention and fault-tolerant
Kafka allows log retention, which means that the messages would be in your system a period of time, as long as your storage resource can fulfill. This would be useful in the following case. For doing some deployment/maintenance, you have to shut down one of your services, for some minutes, or hours. With API, after unsuccessful retries, the client will surely give up. Thus, the request is gone forever. With Kafka, the consumer can restart from the last processed message so your service can start consuming the next message without anything loss.
However, you should not totally depend on the log retention; the log retention can buy us some time but if the rate of consuming message is way below producing, then you should quickly find a way to scale the consuming process, or else there will be a big delay in the system/the message could be lost
### Loose-coupling services
Imagine you have multiple services. When you update one service, you expect others to also be notified and do something on their side. With normal API calls, what you could do is calling each of them individually. However, it would be very manual to add each service into the code. And, what if you were to add other some service later, you have to go back and add code to notify the update to new service, too
How would you handle error here? Failure in one service could prevent the update to other services, and stop the whole system flow. With Kafka, the updated service just needs to notify the new message to the topic, and each service will consume the new message individually. Of course, depending on the use case, we may want to use direct API call instead of Kafka here
### Partition and ordering
In some senses, similar to the load balancer, you can have multiple partitions in one topic and multiple consumers in one consumer group, to divide speed up the processing process. The message will be processed in FIFO order.
If you have multiple partitions, without specifying the key, the message can be going into any partition. This can yield some ordering issues, especially if one message can have a side-effect on the other. In this case, you can specify the key, to ensure these messages would be in correct order.
If you have multiple instances and multiple partitions for one consumer group, you have to ensure that the number of partitions should be the multiple of the number of consumers, so that the message could be (quite) evenly distributed among all consumers. The number of consumers is adjustable, but the number of partitions can only go up. Thus, we should do some calculation here before recklessly increasing the number of partitions, because having a lot of partitions would create more overhead for the overall system

## Setup and maintenance
It is fairly easy to get started with Kafka. You can have it up and running using these two Docker images
```
https://hub.docker.com/r/wurstmeister/zookeeper/
https://hub.docker.com/r/wurstmeister/kafka/
```
However, there would be some maintenance required during operation:
In the production environment, it is also good to remember to have at least 3 brokers, and set the replication factor to at least 2, in case one broker having a problem, the message recovery process would be easier
Sometimes Kafka can go wrong. We were once having a problem on our server, where the Kafka transfer speed is really slow. A restart simply made the problem gone. However, depending on the issue and the application code, we may have lost the message in this case