---
title: "Distributed Tracing and Testing"
date: 2023-02-02T13:16:35-08:00
draft: false
type: post
tags: ["observability", "tracing", "distributed tracing", "testing", "software development", "distributed systems"]
showTableOfContents: true
---

## Intro

Hello again :wave:! With today's post we are going to talk about using
Distributed Tracing and Testing. For this post we will do a quick background
on the concept of Distributed Tracing and use the [Finatra](https://github.com/twitter/finatra)
Open Source project as a way to discuss some of these concepts, but also show
some concrete examples. The example code can be found in my [GitHub examples repo](https://github.com/enbnt/examples.enbnt.dev).


***:headphones: Music Inspiration :musical_note:***

{{< apple_music album="cold-cash-and-colder-hearts/1659728720" >}}

***Jump To***

{{< toc >}}

_Note_

I spent years making contributions to the [Finatra](https://github.com/twitter/finatra) 
project. Following Twitter's acquisition in 2022, I cannot comment as to the continued 
support or usage of Finatra or any of Twitter's other OSS projects. 

While I can't make a recommendation on any new or continued adoption of these OSS projects,
I think that there is a *treasure trove* of discussion that is broadly applicable and
absolutely still valid and relevant. There is so much to mine in simply discussing design
trade-offs, comparisons to other approaches, successes, and pitfalls - this work doesn't just 
exist in a Twitter vaccuum. It has and will continue to inspire other works.

## Background

If you are unfamiliar with the concept of Distributed Tracing, I will do my best to
give a brief background. Much of what exists in industry today can be traced
(pun intended) back to Google's [Dapper Whitepaper (2010)](https://research.google/pubs/pub36356/). This is the opening paragraph:

> Modern Internet services are often implemented as complex, large-scale distributed
> systems. These applications are constructed from collections of software modules that may 
> be developed by different teams, perhaps in different programming languages, and could span 
> many thousands of machines across multiple physical facilities. Tools that aid in 
> understanding system behavior and reasoning about performance issues are invaluable in such > an environment.

&mdash; <cite>
[Dapper, a Large-Scale Distributed Systems Tracing Infrastructure (2010, April)](https://research.google/pubs/pub36356/)</cite>

The 2010's saw an explosion of innovation in this area. This 
[Twitter Engineering Blog from 2012](https://blog.twitter.com/engineering/en_us/a/2012/distributed-systems-tracing-with-zipkin)
discusses [Zipkin](https://github.com/openzipkin/zipkin), 
[Uber's Engineering Blog from 2017](https://www.uber.com/blog/distributed-tracing/) shares
their work in Distributed Tracing and the OSS availability of their project [Jaeger](https://www.jaegertracing.io). 
The concept of Distributed Tracing birthed companies such as [Lightstep](https://lightstep.com) 
and [Honeycomb](https://www.honeycomb.io). The industry seems to be generally veering towards
adoption of [OpenTelemetry](https://opentelemetry.io) as a much needed bridge for all of these
tools.

## Distributed Traces - BUT Local

The primary focus of the Distributed Tracing tools I listed above is on mining and analyzing
the behavior across connected nodes. You need this in order to find things like performance
bottlenecks or to verify behavior of a request in a live, complex distributed system. What
happens when that key trace annotation you thought you were adding doesn't show up? What if
I told you this has happened to myself and (likely many) others? How do you debug and verify
the thing that is meant to help you debug and verify *THE THING*?

## Example Server

```scala
case class ExampleRequest(@QueryParam name: String)
```

```scala
case class ExampleResponse(name: String, score: Int))
```

```scala {linenos=table}
import com.twitter.finagle.http.Request
import com.twitter.finatra.http.Controller

class ExampleController extends Controller {
    get("/") { request: ExampleRequest =>
        // retrieve this request's active trace context
        val trace = Trace()

        // annotate some application specific data to the trace
        trace.record("name", request.name)

        // return the response
        ExampleResponse(request.name, request.name.length * 2)
    }
}
```

```scala {linenos=table}
import com.twitter.finatra.http.HttpServer
import com.twitter.finatra.http.routing.HttpRouter

class MyTracingServer extends HttpServer {

  override def configureHttp(router: HttpRouter): Unit = {
    router.add[ExampleController]
  }

}
```

# Testing via Zipkin UI

The Zipkin UI contains a nifty feature which allows you to upload
a file containing a valid JSON payload with your Trace data to see
a rendering of it in the UI.

```
$ docker run -d -p 9411:9411 openzipkin/zipkin
```

But you can do this by following along with anything in the
[Zipkin Quickstart](https://zipkin.io/pages/quickstart.html).

Then open `localhost:9411` in your browser and you can
click the `Upload JSON` button, which is located next to
the `Search by trace ID` box.