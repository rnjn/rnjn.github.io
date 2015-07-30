---
layout: post
title: "On Postels law and managing change"
date: 2015-07-30 13:49:52 +0530
comments: true
categories: 
---

Postel's law ~1981[^1],

> "Be conservative in what you do, be liberal in what you accept from others." (RFC 793)


A webservice accepting a message with a defined schema (xml/json/other) may choose to do one of these when it encounters messages with extra nodes/properties -

+ be conservative and discard 
+ be liberal and accept

Of the two, there are more proponents for the latter than the former. There are many reasons thrown at the conservatives, one of them that has practical implications is change in schema. Before we go further lets state the obvious - if there is never ever going to be a change in the schema of a message, being conservative is just the same as being liberal. Except that the "never ever" **never** really happens, there's always some change in schema. The choice of being liberal or conservative mandates thinking about how you deal with change in schema, and that's a good thing.

Lets talk about change. Producers and consumers of messages agree on a schema and then continue with their business. When a producer decides to change the schema to improve the featureset, it is generally considered a good practice to communicate the change using software versioning, a version being a quantum of change as agreed. If the responsibility on propagating the change lies with the producer: the producer distributes and manages upgrades of the service client, they need not worry about being liberal. This sometimes is true for custom enterprise software produced and consumed internally. However, this is small subset of software produced these days. Most of the services written today, provide an API and a reference library, and allow consumers to write their own clients.

The implication of being conservative about accepting messages then is that all known and unknown clients of a service have to upgrade in tandem or face downtime, when there is just one version of the service up and running. (Strictness doesn't just mean discard messages with additional properties, it also mandates not accepting deficient messages as per the schema). A better way to propagate change may be to host multiple versions of a service, and that just means **we elevated Postel's law from a producer's view of a particular instance of the service, to a consumer oriented view of liberally providing a service**, albeit via conservative instances of different versions of the same service. The question then is, how many versions of a service would you maintain at the same time? With changes big and small, the number of versions quickly proliferate and cost of hosting and maintenance is not cheap.

The choice of tolerant reader[^2] design doesn't really much help with big changes, since big changes in schema that are needed to service new features etc. need client updates anyway, and can be designed as above. Also, accepting a string that represents the full message, if you were to take Postel's law to an extreme would require a lot of plumbing and would cost higher to work with and make secure. This design choice shines when it comes to adding smaller changes. Lets take a real example. A service that we have been working on receives messages that contain a Doctor's name along with other patient diagnoses related data. A new feature request from end users that we have to implement immediately is to add phone numbers and other contact information of said doctor so that the doctor can be contacted at a later date, drives a change in schema. If we follow the multiple strict service versions approach, we would probably add a new version for this. The cost of managing different versions very quickly starts affecting the quantum of change released to consumers, and producers start designing big releases instead of multiple smaller changes. A liberal design might help contain hosted service instance proliferation by letting older clients send messages they are used to sending, while upgraded clients can send new information too. At a later date, if we for some reason need to downgrade back to a previous version, we do not need to downgrade all clients. This flexibility helps enormously when managing servers and delivering small changes continuously.

Another thing, being liberal isn't easy. A conservative service assumes that schema validation is enough to avoid runtime errors (this assumption is not entirely correct[^3]). A conservative service can assume that it never has to service clients that expect a different version of it. Versions can be deprecated by just not installing them, or removing a server. A service designed to be liberal however can only provide schema guidelines and has to wait to extract all possible information from a message to be able move forward in most cases. Also, a liberal design that enables servicing multiple versions of clients with just one instance of the service needs a lot thought. The producer has to manage change very closely, and since deprecation is a code change, these decisions take more time to production. Its also very hard to keep an eye on how many versions are being supported at the same time. **A combination of a small set of available versions each being liberal to an extent provides a good alternative**. Also, a good way to work on some of the difficulties with liberal reader design are consumer driven contracts.

There is another counterpoint[^4] to Postel's law and liberal reader design - security experts advice us to be conservative in what we accept, contrary to Postel's law. Indiscriminately accepting any message ofcourse has consequences. Yet within the bounds of a sane secure message protocol, a message data schema that allows subelements to be optionally present can be worked with. (Indiscriminately following any advice mostly hurts, including this one).


[^1]: [Postel's law](https://en.wikipedia.org/wiki/Postels_law) - wikipedia
[^2]: Martin Fowler describes decoupled collaboration [Tolerant reader](http://martinfowler.com/bliki/TolerantReader.html)
[^3]: [Schema validation offers a false sense of security](http://blog.iancartwright.com/2006/11/schema-validation-offers-false-sense-of.html) - Ian Cartwright
[^4]: Eric Allman's counterpoints to the principle [The robustness principle reconsidered](http://cacm.acm.org/magazines/2011/8/114933-the-robustness-principle-reconsidered/fulltext)
