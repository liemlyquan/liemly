+++ 
draft = false
date = 2018-03-15T07:00:00+07:00
title = "Finding the right bottleneck"
description = ""
slug = ""
authors = []
tags = ["optimization", "aws"]
categories = []
externalLink = ""
series = []
+++


One of my favorite area of software engineer is performance tuning. No matter which kind of software I am writing, a desktop/mobile/web app, or even a script, I always want it to super fast, with as little resource as possible. Yet, there have been some occurrences that I have wrote/failed to detect some bottlenecks that makes me closer to my wishes.

In university, when I was doing a website development for event, we run into free-tier bandwidth limit of Google App Engine after a few hours when the number of users spike. I tried to optimize in many ways, including but not limited to: minify source code, optimize process, cache. In the end, I found out that there is one single image that I had not optimized, and it was on the main page of the website. Thus, each main page loading in fact takes MBs of bandwidth. After finding out the problem and optimizing the images, the bandwidth never go over 25% again. In another word, ~8 times less bandwidth used.

Another example would be when I was working on optimize our initialization at Chopp. The initialization endpoint at that time would take approximately 2–4s to respond. When we are requesting to the endpoint, there were about >10 processes to gather data from different sources. At first, we (I and Tuan Nguyen) were planning to split up this process, so the first initialization will be faster. Yet, what amazes us is that, we found out a single process that takes up close to 100% of the total time. We went on to identify the problem with that process, and have it fixed in one night. After that optimization, the initialization endpoint response reduced to 400–500 ms (Postman). It would be 4–8 times faster.

The last example would be an incident with AWS. With the expansion our whole ecosystem, we have seen an ever-increasing price in the billing. recently, we decided to tackle with this one. There were several small things that we have done: removing the unused resources, making change to instance type. Still, among all the changes, there is ONE that is equal to all others combined. In our Redis, we have set the wrong expire date for one type of data. This takes our 10 times more storage than what we need. By setting a more proper expire date, and modifying that instance, we have reduced our total AWS cost by 1/4 :)

Of course, there would be many required optimization among all the systems we are all working on. What I learned from all of the lessons are that, quite often, there are few optimization that can have significant improvement on our whole system (80–20 rules ?). With the limited time and resource, we should identify and solve those make real impact, then the leftover would be insignificant.