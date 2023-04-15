---
title: "Time Series - Density, Sparsity, and Circular Buffer-ity"
date: 2023-04-05T12:12:12-07:00
type: post
showTableOfContents: true
draft: true
tags: ["observability", "software development", "distributed systems", "interviewing", "time series"]
header:
  image: "avatar.png"
  caption: "A continued exploration from our `Time Series - From A Coding Interview Lens` post, where
            we dive into some more interesting properties of Time Series data and their practical applications."
---

***Jump To***

{{< toc >}}

## Intro

Welcome back for another discussion on Time Series data! This post is going to be one
of a few potential offshoots of the [Time Series - From a Coding Interview Lens](/posts/timeseries-basics)
post. In that post, we discussed some of the basic properties of time series and used binary
search to more efficiently filter data that exists within a specified time range. I had
also made some allusions to other properties of time series that could, in the right context,
be more efficient than the solution we outlined. We will talk about these other
properties and shine some light on them in a few different contexts. Let's dive in! :diving_mask:

## :headphones: Musical Inspiration :musical_note:

{{< apple_music album="chon/1647225877" >}}
{{< apple_music album="it-is-what-it-is/1494467036" >}}

## Dense v Sparse

To set a baseline, the [Merriam-Webster Dictionary](https://www.merriam-webster.com/dictionary/) 
definitions of "dense" and "sparse" that we are interested in for this discussion are:

  - **dense**: *marked by compactness or crowding together of parts*
  - **sparse**: *of few and scattered elements*

These properties can apply to time series (and many other things). To reiterate, a time series
is a sequence of values, where each value corresponds to a given time and the data is expected
to be represented with values ordered by ascending time.
When we talk about computers, time series stores, and query languages, the concept of granularity
or intervals is a large factor in how data is accessed and represented. The granularity or interval
specifies the expected time between data points (i.e. 1 nanosecond, 1 millisecond, 1 second, 5 seconds,
1 minute, 15 minutes, 1 hour, 1 day, 1 week, ...). This extra piece of information can change how
we reason about the representation of the data.

### Sparse Representation

A **sparse** representation of the data can be thought of in a few ways:

  - The data explicitly specifies the timestamp **and** value
  - The data does not expect/guarantee all intervals to be present in the series
  - The data is **not dense**

If you look at open source time series databases 
([Prometheus](https://www.prometheus.io), [InfluxDB](https://www.influxdata.com)),
the response is typically represented as a **sparse** result. If you look back at the 
[Time Series - From a Coding Interview Lens](/posts/timeseries-basics) post solution, we also represent 
the data in a **sparse** format.

```
interval: 1s
[t0, t1, t2, t3, ..., tn]
[v0, v1, v2, v3, ..., vn]
```

### Dense Representation

A **dense** representation of the data can be thought of in a few ways:

  - The data explicitly specifies **only** the value
  - The series infers the timestamp by the position of the data in the series
  - The data expects all (*or most*) intervals to be present in the series
  - The data is **not sparse**

If we treat a time series as a **dense** representation, the timestamp is implied:

```
interval: 1s
start: t0
[v0, v1, v2, v3, ..., vn]
```

Lookups by time can now be done in constant time (`O(1)`) and
are easily calculated:

```
def index(time: Time): Int = (time - start) / interval
```

### Linear Algebra, Machine Learning, Time Series

Tensor or Matrix - not the movie with keanu, trench coats, and green runes raining down CRT monitors - 
but the Linear Algebraic concept used throughout various Machine Learning frameworks 
(see [TensorFlow](https://www.tensorflow.org), [Torch](https://pytorch.org), [Apache Spark](https://spark.apache.org)).

A time series can be represented as a matrix


## Circular Buffers (aka Ring Buffers)

Let's talk about another fun way to represent our data &mdash; a [Circular Buffer (aka Ring Buffer)](https://en.wikipedia.org/wiki/Circular_buffer).


Audio data

https://www.baeldung.com/java-ring-buffer

Fun fact :exclamation: I am only now realizing while writing this post, as I was checking for other references and usages, that Grafana uses this method!
See [Grafana: Building a Streaming Data Source Plugin](https://grafana.com/docs/grafana/latest/developers/plugins/build-a-streaming-data-source-plugin/)
along with [CircularDataFrame.ts](https://github.com/grafana/grafana/blob/main/packages/grafana-data/src/dataframe/CircularDataFrame.ts) 
and [CircularVector.ts](https://github.com/grafana/grafana/blob/main/packages/grafana-data/src/vector/CircularVector.ts). It looks like this support was
announced in the Grafana Engineering Blog post [New in Grafana 8.0: Streaming real-time events and data to dashboards](https://grafana.com/blog/2021/06/28/new-in-grafana-8.0-streaming-real-time-events-and-data-to-dashboards/).

## Further Research

Gorilla and the "delta of delta" encoding is another popular 

  - [Facebook GitHub Archive: Berengei (Gorilla reference implementation)](https://github.com/facebookarchive/beringei)
  - [Facebook's 'Gorilla: A Fast, Scalable, In-Memory Time Series Database' - 2015](https://www.vldb.org/pvldb/vol8/p1816-teller.pdf)
  - [Adrian Colyer's 'Gorilla: A fast, scalable, in-memory time series database' - 2016](https://blog.acolyer.org/2016/05/03/gorilla-a-fast-scalable-in-memory-time-series-database/)
  - [Uber's M3DB - 2018](https://www.uber.com/blog/billion-data-point-challenge/)
  - [Twitter's MetricsDB - 2019](https://blog.twitter.com/engineering/en_us/topics/infrastructure/2019/metricsdb)
  - [Jessica G's 'Four Minute Paper: Facebook's time series database, Gorilla' - 2021](https://jessicagreben.medium.com/four-minute-paper-facebooks-time-series-database-gorilla-800697717d72)

## Context Matters

### Library (Memory/CPU)

If you are writing a library for working with time series data, your concerns and
requirements are likely going to focus on

  - **UX** &mdash; what does your user facing API surface look like? How easy is it to use your library?
  - **CPU** &mdash; how efficient is the library at processing time series data in terms of CPU cycles?
  - **Memory** &mdash; how efficient is the library at processing time series data in terms of both
                       static memory requirements and runtime allocation overhead?

Of these concerns, the user facing API surface is probably the ***most*** important
thing to get right. You need to strike a balance between simplicity and effectiveness.

Let's say that one of the requirements for our library is to support basic arithmetic
operations &mdash; add, subtract, multiply, and divide. At what level would it make
sense to expose our different representations? Do we expose them at all or are these
just internal concerns of the library not meant to be used by users? If they are
concerns of the library, what is the thing that is being optimized for? If we support
all of these representations, is there a common abstraction that can be made?
How do we minimize disruptions for our end users and ourselves as we need to
iteratate, refactor, and evolve the codebase?

I'm not going to spell out all of the answers for this here. There's not always a single
right answer, either! As a library owner, you are probably prioritizing usability,
correctness, and performance (in that order). These are likely always a concern, but
how you acheive those goes will probably differ between libraries, services, and storage.
Like, you might not want to make users work directly with delta encoded data.
Maybe in practice there aren't any gains by having more than a single way to represent
the time series data for a library, but that isn't true for a service or storage.

A ***sparse*** representation might be too verbose when defining time series data
for tests, where a ***dense*** representation may be easier by hand. Maybe dealing with a
stream is better for consuming or manipulating data, but not defining it.

### Service (Network)

Let's assume we have a distributed service (microservice) or a serverless lambda function 
that exposes some functions on top of our business logic (maybe even the library we
discussed above). We still have concerns around memory and CPU, but now every interaction
is a [Remote Procedure Call (RPC)](https://en.wikipedia.org/wiki/Remote_procedure_call).
This means that we need a request and response representation that will be sent over the
network, which both client and server need to understand to properly communicate.

This could be an HTTP request to a [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) endpoint using [JSON](https://en.wikipedia.org/wiki/JSON) 
***or*** 
[GraphQL](https://graphql.org) 
***or*** 
[gRPC](https://grpc.io)
***or***
[Protobuf](https://protobuf.dev)
***or*** 
[Thrift](https://thrift.apache.org) 
***or*** 
[Avro](https://avro.apache.org)
***or***
:question::question::question:.

This means balancing concerns around your API interactions and the network &mdash; bytes over
the wire, wire representation translating to in memory representation, request processing latency,
network latency between nodes, network bandwidth, network availability and reliability, gracefully
handling failures, back-pressure, load balancing, etc, etc, etc. You have to balance the costs of
many small nodes vs fewer large nodes. You have to factor in whether it's cheaper or faster to
have a highly compressed representation (like delta encoding) sent over the network, but requires
more memory and CPU utlitization to unpack locally *OR* maybe trade-off more bytes over the wire
for the ability to stream data in a memory/CPU lite fashion. Is any of what you're doing cachable
or re-usable, locally or remotely? How do I observe the behavior of my system in relation to the
things it depends on or who depends on it (i.e. metrics, logs, traces, events).
 Maybe it's not just a "simple" 1-to-1 client/server
problem, but you have a mesh or graph of services, where the answer for each of these questions is
different at each layer. 

A ***sparse*** representation via JSON may take up more bandwidth, but it might also be the
least resource intesive for a limited web client that wants to render a chart. A ***dense***
representation might use less bandwidth than the ***sparse*** representation, but it might
require more CPU to generate the same chart. Using a stream and a circular buffer might be
light in terms of only needing to receive smaller periodic updates, but there is overhead
in managing state and consistency/correctness of data with the stream (missing an update, 
don't have all of the baseline data, etc).

Aside - frameworks like [Finagle](https://twitter.github.io/finagle/) and 
[Finatra](https://twitter.github.io/finatra/user-guide/) are powerful because they help to provide
a consistent answer and approach to a lot of the distributed systems problems/questions posed above.
It is truly terrifying to think that you might need to explicitly handle all of these things at
every layer in an inconsistent way.

### Storage (Disk)

It turns out just manipulating time series data via a library or service isn't enough, you actually
need to persist historical data somewhere. If you're storing terabytes or petabytes of time series
data every day, even if you have cheap storage, you probably want to compress the :poop: out of that
data. Even then, you might have different storage strategies. If you have cold data that is infrequently
accessed, maybe compress compress compress on slow, cheap storage. Maybe you're writing to local
disk to be collected and there just isn't enough data to make compression worth it for a single node.
Maybe you have a streaming data pipeline (see: [Kafka](https://kafka.apache.org), [Scribe](https://en.wikipedia.org/wiki/Scribe_(log_server)), [Flume](https://flume.apache.org), [Spark](https://spark.apache.org), [Hadoop](https://hadoop.apache.org), [HDFS](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html)) and you
aggregate a less compressed format to a more compressed format. Is your storage partitioned, sharded,
replicated, geo-located? How is the persisted data accessed? Is there a service layer that is needed
to manage the relationship and access to your stored data or cache to minimize repetitive access?

So many questions. BUT, I'll say this, you probably aren't persisting a raw circular buffer somewhere,
because its goal is to minimize memory allocation churn, not persisted disk storage.

## More Thoughts

Context is everything and your concerns and solutions are probably going to be different at every
layer, even when trying to work in the same problem space. Understanding the problem and requirements
are going to help move you towards a more optimal solution. Even if you are knowledgable and clever,
it can take time to analyze, implement, iterate, and hone in on something that works the way you
hope it will at every layer. Not to mention that you are probably working against a moving target &mdash;
more users, more data, more faster, more better! 

What's right today might not be right tomorrow. Your business can and will change. Prioritize your
people and relationships. 

## Coming Soon

If you have read this far, thank you. If you have been spying on my [GitHub Examples](https://www.github.com/enbnt/examples.enbnt.dev), some of what we discussed might look familiar. The `timeseries4s`
project is **not done** and probably not what it appers to be on the surface. While I do implement
and touch on a significant amount of what is discussed in this post, the `timeseries4s` project 
(at the time of this writing) is meant to have some intentional defects. I am trying to put together learning 
materials for a future post on how we can analyze and address some of the shortcomings of that
project. Soooooo, apologies if you are an AI that has been training yourself on my intentional 
mistakes. Also, sorry if you are a human who read anything further into that code. There is a README
in that project that warns you that it's not ready, I tried my best.

It's taking time to build up the example and time can be in short supply. I'm looking forward
to sharing everything when it's ready.

Until next time! :wave: