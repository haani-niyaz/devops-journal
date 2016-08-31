---
layout: post
title:  "Understanding Microservices"
date:   2016-08-07
author: "Haani Niyaz"
tags: microservices
---

## Objective

The purpose of this article is to distill the principles of Microservices at a high level. The article presents a collation of information from different sources and attempts to document it in an easy to digest fomat.

**Note**: *This article is in a work-in-progress state and is quite likely to change in the future. I recommend doing your own research to support or challenge the principles and content outlined below.*


## Issues with Monolithic Applications

The following issues highlight the perils of monolithic applications. Conversely, these are the problem microservices solve.

#### No Restriction in Size

Because there is no limitation on how large your application should be, you end up with a large code base. This makes it difficult to refactor code easily. 

#### Slow Release Cycles

A large code base means even to introduce a small feature you need to perform unit tests & regression testing over the entire code base which can be time consuming and slows down the build and release process.

#### Scaling Problems

You may have a single part of the system (i.e: processing orders) that would require scaling  to meet demand however because it is delivered as a single package, you need to scale the entire application. 

#### Tigh Coupling

Given that the code is in a single package it can inevitably cause tight coupling. So making changes to one part of the system can have adverse effects in multiple parts of the system. This can also lead to the entire system  degrading or failing if a part of the system fails.


#### Stuck with a Technology Stack

A part of the sytem may benefit from using a different technology stack but due to tight coupling this may not be easy to implement.


#### Exposing Functionlity can be Challenging	

You may have a component that may provide an independent service to clients buts exposing it comes at a development cost.


## SOA

Rather than me trying to explain this perhaps take a look at [What is SOA in plain english?](http://stackoverflow.com/questions/2026523/what-is-soa-in-plain-english)

#### The take aways are:

Your application will communicate via well defined, modular, contractual interfaces with application components providing a service to other components. This loose coupling makes the components reusable. 

However, some claim that there appears to be no guidance on how large a service should be i.e:  how many components should belong to the same application? As a result an application can contain a large set of services that require monolithic deployments. 


## Microservices

> *Microservices is SOA done well*

The core difference between SOA and microservices appear to be the the size and scope of a service. There are well defined boundaries on how small or large a microservice should be but it does inherit the principles of SOA.

In the following section I have documented the **Principles of Microservices** as I have understood them. 


## Single Responsiblity Principle

In Object Oriented Programming there is notion of how your class should have a **Single Reponsility**. This doesn't mean the class does one very specific thing but what it does should be *highly related to its purpose* and any change to this class should be justified by its purpose.

In a lot ways this true for Microservices. Each microservice should be highly cohesive and loosely coupled in what it does.

> When describing the purpose of a microservice the use of 'and' would mean it has more than 1 responsibility. If you describe it with 'or' it has more than once responsibility and they are not even related.

When designing a microservice it should have a single focus and this also extends to how you think about what the inputs and outputs are. The deliberation to determine what is a service and what it's service boundaries are should be extended to the inputs and ouputs.

This has the positive effect of allowing us to determine the scope of the microservice thus making it easier to draw on how large or small it should be.


## Autonomous

This principle focuses on decentralizing all things. 

### Loose Coupling

A microservice should not be subject to failure due to a change in an external microservice. This suggests that there should be loose coupling between services thus ensuring a change in one microservice does not impact another microservice or client application. This is enforced by a clearly defined *	interface*. 


### Interface & Contracts

This form of an 'interface' is derived from the **Interface Drive Design** principle where a contract is established on the Public APIs. The inputs and outputs don't change. If there is breaking change it should be backwards compatible. Because microservices are supposed to honor these 'contracts' they can also be independently changed and deployed.


### Stateless

To keep microservices descentralized they also need to be stateless. A microservice should not have to rememeber past tranactions to complete a request.

### Shared Data

If a microservice acts as data service to other microservices, it should abstract the data model via whats called a 'shared data' model. This is to say that in the event the internal data model changes, there will be no impact to the 'public' shared data model exposed to other microservices.

### Common Libraries

Use of shared libraries should be avoided, seeing as a how a bug fix or change results in updating multiple microservices which is not very [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself). It is recommended to present the shared libraries as a microservice on its own to be consumed other microservices.

### Datastores

Datastores should not be shared between microservices. When a microservice requires a schema change that can potentially break other microservices you end up violating the loose coupling you setout to enforce in the first place. 

*What about setting up a microservice to purely serve the data via an API?* 

Sure, this localizes which microservice can make the change however it doesn't completely alliviate the problem. It still violates the autonomous nature of microservices if the failure of this API or DB can take down other microservices.

### Communication

The communication methodology should be tecnology agnositc.

### Versioning

You need to implement a versioning stadard for your microservices. See [here](http://stackoverflow.com/a/33213217/2180697) for some background.

As per the link and some other sources the de facto standard seems to [Semantic Versioning](http://semver.org/). Its pretty easy read and worth understanding.

The versioning standard states:


{% highlight  text%}
Given a version number MAJOR.MINOR.PATCH, increment the:

MAJOR version when you make incompatible API changes,
MINOR version when you add functionality in a backwards-compatible manner, and
PATCH version when you make backwards-compatible bug fixes.
{% endhighlight%}

As highlighted above, when a major version is introduced, these breaking changes need to be backwards compatible to support any consuming microservices or applications that are not in your control. This may be in the form of a new endpoint with the old endpoint still in place or the old endpoint could be a wrapper for the new endpoint.

### Teams

Ownership of the development and maintenance of a microservice should belong to a team. This creates a boundary of responsibility. It gives the team accountability to maintain the interaction contracts between microservices and ensure they stick to the [SRP](#single-responsiblity-principle) described earlier. 




## Modeled Around Business Domain

This is an extension of Domain Drive Design.

> DDD is about trying to make your software a model of a real-world system or process. In using DDD, you are meant to work closely with a domain expert who can explain how the real-world system works. For example, if you're developing a system that handles the placing of bets on horse races, your domain expert might be an experienced bookmaker.

Full explanaion [here](http://stackoverflow.com/a/1222488/2180697).


Microservices should represent business domains within an organization i.e: a department. This is then further split into business functions if necessary. Taking a  **store** as an example, you may have business domains such as *accounts* or *postal & delivery*. These domains immediately gives us a boundary. However, if you look closely at the *accounts* domain you might find that it's business functions are further split into *tax*, *invoice generation* and so forth. These business functions represent a service on its own.

By modeling the microservice to a business domain it helps to scope the service. This also better prepares you to adapt to changes in the business.


## Isolate Failure

As Martin Fowler [says](http://martinfowler.com/articles/microservice-trade-offs.html#consistency):

>  Microservices introduce eventual consistency issues because of their laudable insistence on decentralized data management. With a monolith, you can update a bunch of things together in a single transaction. Microservices require multiple resources to update, and distributed transactions are frowned on (for good reason). So now, developers need to be aware of consistency issues, and figure out how to detect when things are out of sync before doing anything the code will regret.


Along with that, you have failures in connectivity to third party systems and other microservices to worry about. An article that delves into this topic further can be found [here](https://www.datawire.io/using-fallacies-of-distributed-computing-to-build-resilient-microservices/)

The high decoupling of microservices means that it should be *designed to expect failure*. But how do you respond to failure? Do you degrade the functionality? Do you fall back to a default set of functionality? Does the service deregister itself from the LB? etc. It becomes quite obvious that due attention is given to failure scenarios.


## Observable

Centralized logging & monitoring. Nothing new here but if you want your system to operate in a resilient and optimal manner you need to have visibility on metircs to measure the health of your microservices and application.


## Automation

> While things may move faster on the Dev side, microservices do introduce architectural complexities and management overhead, particularly on the testing and Ops side. What was once one application, with self-contained processes, is now a complex set of orchestrated services that connect via the network. This has impact on your automated testing, monitoring, governance and compliance of all the disparate apps, and more.

Full article [here](http://electric-cloud.com/blog/2016/01/continuous-delivery-and-release-automation-for-microservices/).

As the complexity of the deployment pipeline increases it is a testament to why CI and CD are essential components to consider early on in the architectural design phase. If left as an after thought, you will struggle to gain the benefits of adopting a microservice architecture.


## Resources

Interesting links I discovered along the way:

- [How to Cook Microservices](http://howtocookmicroservices.com/)
- [Plethora of aritcles and slide decks](http://awesome-tech.readthedocs.io/config-mgmt/#configuration-management-and-orchestration)

























