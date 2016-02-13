---
layout: post
title: "SOA on Rails: Part 1"
date: 2016-02-09 15:40:45 -0700
comments: false
author: Rahmal Conda
categories: SOA, Rails, Ruby, Web Services
---

# SOA on Rails: Part 1 - Intro

> *“When you hit the Amazon.com gateway, the application calls more than one hundred services to collect data and construct the page for you.”* **- Werner Vogels, Amazon CTO**

## Amazon's Mandate
About 15 years ago, back when Amazon was just an online bookseller, Jeff Bezos, Founder and CEO, gave every team at Amazon a mandate: ***To rebuild Amazon’s infrastructure into a Service-Oriented Architecture from top to bottom. but this is around 2001.*** The term SOA hadn’t even been invented yet. So what exactly did Bezos ask his teams to do? This is the mandate as expressed by Jeff Bezos:

    1. All teams will henceforth expose their data and functionality through service interfaces.

    2. Teams must communicate with each other through these interfaces.

    3. There will be no other form of inter-process communication allowed: no direct linking, no direct reads of another team’s data store, no shared-memory model, no back-doors whatsoever. The only communication allowed is via service interface calls over the network.

    4. It doesn’t matter what technology they use.

    5. All service interfaces, without exception, must be designed from the ground up to be externalizable. That is to say, the team must plan and design to be able to expose the interface to developers in the outside world. No exceptions.

Bezos end his mandate with this: ***Anyone who doesn’t do this will be fired.  Thank you; have a nice day!***

Even if they couldn’t guess what Bezos’ mandate would mean for them at the time, No one can dispute the results. Nor can anyone deny that Bezos was years ahead of the curve. Does anyone think of Amazon as a bookseller now?

## What Came Before
### Big Bad Monolith (The Ol’ Days)
A few years after Bezos’ Mandate, a small band of opinionated ruby developers were releasing Rails into the ether and introducing Ruby to the ways of the distributed system and the enterprise architecture. Their first attempt didn’t break any records, but through a small but dedicated (and growing) following. Rails muscled its way into the mainstream. Most Rails apps at the time were small MVC stacks on top of a single database, on which the application depended heavily. This was a big flaw that was rather obvious, but forgiven because of Rails’ seemingly magical ability to produce a complete, functional, though a bit brittle) web stack with a trivial amount of time and effort.
 
<center>![Monolith 1][monolith1]</center>

Don't get me wrong. The Rails community, along with developers of *Ruby Gems* and of other related technologies such as *Passenger (ModRails), Nginx, Memcached, PostgreSQL, MySQL*, and many others worked tirelessly worked to make Rails the center of a scalable multi-tiered platform worthy of even the most sophisicated application architectures. With the help of Load balancers and other scaling strategies, many developers have found success with more traditional Rails stacks.

<center>![Monolith 2][monolith2]</center>

Common Rails deployments, even those that have not fully embraced SOA, have service-oriented modules within the web and application layers. For instance, Rails controllers conventionally implements RESTful endpoints for their resources. And ModRails has favored a threaded approach to allow many app instances to run concurrently. However, encapsulation has been mostly a province of the code layer.

Don’t feel bad if your app still looks like that. Everyone starts out like that. The Important thing is that you are reading this. Because if you are, it means the time has come to take this to the next level, and you’re ready to act. Let’s go build a web service! 

<center>*...Hold on… What exactly is a web service anyway? I’m glad you asked!*</center>

## What is a Web Service
### Working Behind The Scenes
Web services are self-contained, modular, distributed, web applications that can be described, published, located and/or invoked over a network to execute some action specified in the request. These applications are ideally small, representing a single, specific function or domain model. Distributed over a network and usually web-based, they can be internal, with some managing application as the primary consumer or other services acting as clients. They can also be external and web-facing, accepting requests from 3rd-party client applications (Facebook API, Twitter, and Google Maps come to mind). Regardless, they are all built on top of open standards such as TCP/IP, HTTP, JSON, and XML. Well, that’s what I heard anyway. 

<center>![Web Service][service1]</center>

### API (The Contract):
In order to provide the service for which it is intended, a web service needs several things. It needs a publisher or broker so clients can find it. But knowing where to send the envelope is only the beginning. Clients also need to know how to communicate with the service. The Service Contract is the means by which a service communicates its API to potential clients. An Application Program Interface (API) specifies how software components should communicate and interact. It describes the location, scheme, protocol, and functions used by a service to accept and process requests, as well as prepare and return responses. 

### Messaging (The Transaction):
The client and service talk to each other via messages. Clients send a request to the server, and the server replies with a response. Apart from the actual data, these messages also contain metadata (HTTP Method, headers, etc.) about the message. The structure of the message depends in large part on the protocol the service implements.

<center>![Web Service Messaging][service2]</center>

The two most common protocols are SOAP and REST (with REST emerging as the clear frontrunner in later years). The figure below shows a simple RESTful web service using JSON as the message format. I’ll get into them more later, but I’ll summarize the difference like this:

**SOAP** - A soap-based web service has an explicit service contract in the form of a wsdl document. A wsdl file contains a very detailed description of the service’s API. Given the wsdl file, a client will know exactly how to interact with the service, including the data is needs, and the format of the request and response. It’s explicit nature gives the service developer much more control over how the service accessed, and in what context.

**REST** - A RESTful web service is the triumph of convention over configuration, given that its contract is implicit. It provides no detailed description, because the request’s envelope is the contract. To be RESTful, a service must adhere to the HTTP protocol and its resource-based methodologies. Most would say that what REST sacrifices in control, it more than makes up for in simplicity and maintainability.

<center>![SOAP vs REST][service3]</center>

## What is a Services-Oriented Architecture (SOA)
### Concepts and Methodology
SOA is a distributed software architecture designed to facilitate the communication, interaction, and collaboration of loosely-coupled, self-contained web services. Like traditional multi-tiered architectures, SOA is based on a strategy wherein software components are distributed across a network. SOA, however tries to represent business processes and domains as shared, reusable components that can be combined in different ways. Service-orientation takes much of its inspiration and many of its principles from object-orientation. Just as with object-orientation, concepts such as encapsulation, abstraction, and reusability are fundamental to the composition of services within the SOA platform. 

<center>![Simple Stack][stack1]</center>

### Separation of Concerns:
The concept described by the phrase Separation of Concerns, is essentially a methodology that attempts to break large applications into a set of individually defined functions or "concerns." The logic required to address or solve the larger problem can then be broken down into individual units of logic that address specific concerns. Many design patterns and distributed architectures have applied this concept in different ways. SOA has uniquely evolved out of an ambition to realize a separation of concerns among stand-alone services that can then be accessed and reused by multiple applications. For example, an “Authentication” service can be used for single-signon into numerous applications. Or a “PDF Generation” service can be used by a banking platform to generate invoices or render banks statements. Hell, we could spend all week coming up with ways an email scheduling service could be used any application… but I digress

### OUR SOA Platform (At a Glance)
While 3rd-party provider services are well-known, Grand Rounds has mainly implemented the internal services. We maintain several domain-driven web services, handling requests from our main web application and other existing web services. We do employ some provider APIs, such as bulk mail and analytics, for example, but it is the internal type I will be discussing in this article. All of our services are self-contained, compact Rails applications. The only exceptions are a few Sinatra apps running internal, non-critical processes. Our frontend UI and our mobile applications are clients to these services. Some talk directly to the clients returning data and other resources, but most are asynchronous, and event-driven in nature. A high-level, extremely simplified view our system would look like this:

<center>![Our Stack][stack2]</center>

## Why SOA
### Benefits:
Let’s take a look at a simple SOA architecture. Then we can see what benefits we can glean by comparing the Monolithic approach we started with. Here’s our platform. What have we gained by moving to this approach. 

<center>![SOA Stack][stack3]</center>

Each service becomes a surprisingly simple component of a complex application platform. Independently, they focus on a single task, but together they form an aggregate application that is both fast and durable. Able to carry larger loads, and yet absorb more losses. The termination a single worker process or a single server instance doesn’t cripple the larger application as a whole.

### Growing Pains:
The first and most obvious benefit of Our new architecture is redundancy. With a system like this we can start eliminating those single points of failure from the first approach. Instantly, we have a more robust architecture. In the case of the Database, it is best practice to setup redundancy through methods such as master/slave replication, sharding, etc. 

### Encapsulation:
Now we can pulling critical domain paths out of our monolithic Rails application, and moving them into independent services. Think about it this way: If Service A represents authentication/authorization service and Service B is a PDF generating service, both having been lifted as separate domain functions from your original application, then now you can update PDF generator engine without affecting the auth service at all. You can move PDF generation to a 3rd-party service without changing the rest of the application. Neither of these things were possible in a single, massive Rails app. As you add or update features in your application, you risk introducing bugs or worse into the rest of the application. Changing anything might break everything. But best of all, the very concept of an SOA architecture makes it easy to move over to a full SOA platform in careful, compact steps. The flexibility it give you allows you to move critical domain logic into self-contained services a peice at a time.

### Scalability:
I’d say the most important reason is scalability. Each of our services can be scaled independently. In the old days our public UI and our internal application was ran on the same server instances and bound them to live and die together. By making them separate services we were able to adjust the resources given to each separately. That goes for all of our other services. As you grow and begin to get more traffic, scalability will ascend your priority list and stay there.

### Next:
Now that the *What* and the *Why* are clear. Let start putting the pieces together. 

**SOA on Rails: Part 2** *- SOA Concepts in Rails*


[monolith1]: https://s3.amazonaws.com/grnds-uat-blog/soa/monolith01.png "Rails Monolith 1"
[monolith2]: https://s3.amazonaws.com/grnds-uat-blog/soa/monolith02.png "Rails Monolith 2"

[service1]:  https://s3.amazonaws.com/grnds-uat-blog/soa/service01.png  "Web Service 1"
[service2]:  https://s3.amazonaws.com/grnds-uat-blog/soa/service02.png  "Web Service 2"

[stack1]:    https://s3.amazonaws.com/grnds-uat-blog/soa/stack01.png    "SOA Stack"
[stack2]:    https://s3.amazonaws.com/grnds-uat-blog/soa/stack02.png    "GR's Stack"
[stack3]:    https://s3.amazonaws.com/grnds-uat-blog/soa/stack03.png    "SOA"