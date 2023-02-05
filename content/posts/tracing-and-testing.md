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

{{< apple_music "freeway-jam/158639291" >}}

***Jump To***

{{< toc >}}

_Note_

I spent years making contributions to the Finatra project. Following Twitter's
acquisition, I cannot comment as to the continued support or usage of Finatra
or any of Twitter's other OSS projects. I think the content and discussion are
still valid/relevant, as we can discuss things like design trade-offs or how
this work relates to or can inspire other works.

## Background

If you are unfamiliar with the concept of Distributed Tracing, 

[Zipkin](https://github.com/openzipkin/zipkin)
[OpenTelemetry](https://opentelemetry.io)

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