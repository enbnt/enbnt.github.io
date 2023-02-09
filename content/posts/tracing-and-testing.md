---
title: "Distributed Tracing and Testing"
date: 2023-02-10T12:12:12-00:00
draft: true
type: post
tags: ["observability", "tracing", "distributed tracing", "testing", "software development", "distributed systems"]
showTableOfContents: true
---

***Jump To***

{{< toc >}}

## Intro

Hello again :wave:! With today's post we are going to talk about using
Distributed Tracing and Testing. For this post we will do a quick background
on the concept of Distributed Tracing and use the [Finatra](https://github.com/twitter/finatra)
Open Source project as a way to discuss some of these concepts, but also show
some concrete examples. If you want to just dig through the full code I based this post off of,
you can find it in my [GitHub examples repo](https://github.com/enbnt/examples.enbnt.dev).


## :headphones: Musical Inspiration :musical_note:

Some of the music that helped me get through this post. Especially getting Bazel working with 
Scala and Finagle in order to write and validate the example code: 

{{< apple_music album="cold-cash-and-colder-hearts/1659728720" >}}

{{< apple_music album="blow-by-blow/158639291" >}}

:broken_heart: RIP Jeff Beck :broken_heart:

## A Quick Note / Disclaimer

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
the behavior across distributed, connected nodes. You need this in order to find things like performance
bottlenecks or to verify behavior of a request in a live, complex distributed system. What
happens when that key trace annotation you thought you were adding doesn't show up? What if
I told you this has happened to myself and (likely many) others? How do you debug and verify
the thing that is meant to help you debug and verify *THE THING*?

Enter Finatra's [InMemoryTracer](https://github.com/twitter/finatra/blob/finatra-22.12.0/inject/inject-core/src/test/scala/com/twitter/inject/InMemoryTracer.scala).
This utility integrates into the Finatra framework's test utilities, which allow for standing
up a full server instance, including its network stack, and putting that under test in a 
very streamlined fashion. I also somehow found a way to create an example without needing to
delve into concepts like asynchronous programming with Futures or Dependency Injection.


*Aside: While the InMemoryTracer feature has existed since May 2022 and is part of multiple releases, 
the OSS documentation has sadly not been updated in GitHub pages. All is not lost! Managing internal
vs OSS builds (and priorities) is tricky. The point of this post is to discuss the concept, we're good.*

In order to discuss the concept, let's look at some simple examples, as referenced
from our [tracing-and-testing](https://github.com/enbnt/examples.enbnt.dev/tree/main/tracing-and-testing)
example project.

### Controllers

All of Finatra's routes and logic are defined via Controllers that are installed via
the server definition. What you need to know from this is that we're defining a route
at path 'GET /score' with some nonsense example logic to illustrate how or why we might
need to test traces in our local business logic... locally... 

If the request has a "name" query param, it will return a score that is the length
of the "name" string (i.e. `GET "/score?name=ian`). Unless there is a "multiplier" 
integer value supplied as another query param on the request (i.e. `GET "/score?name=ian&multiplier=5`),
the score returned will be the 'name' length multiplied by the 'multiplier'.
If no 'name' query param is present, a -1 is always returned as the score.

The point of this example isn't the logic, it's that we have a way to conditionally
append an annotation to our distributed trace. It's pretty common to have branching
or conditional logic and only want/need to annotate in those scenarios. This is
accomplished via our `trace.recordBinary` calls in the controller.

```scala
class ExampleController extends Controller {

  get("/score") { request: ScoreRequest =>
    val ScoreRequest(optName, optMultiplier) = request

    val score: Int = optName match {
      case Some(name) =>
        // retrieve this request's active trace context and
        // annotate some application specific data to the trace
        val trace = Trace()
        if (trace.isActivelyTracing) {
          trace.recordBinary("example.name", name)
        }

        val mult = optMultiplier match {
          case Some(m) =>
            if (trace.isActivelyTracing) {
              trace.recordBinary("example.multiplier", m)
            }
            m
          case _ => 1
        }

        name.length * mult

      case _ =>
        -1
    }

    // return the response
    ScoreResponse(optName, score)
  }
}
```

Here is another example Controller, which uses the concept of Process Local Tracing.
This is a way to include information in your distributed trace that is key to understand
within the local process node's execution. This example is meant to illustrate critical lifecycle
phase timings:

```scala

class LifecycleController extends Controller {

  post("/lifecycle") { _: Request =>
    val trace = Trace()
    trace.traceLocal("example.lifecycle.init") {
      // initialize and prep for work
      trace.traceLocal("example.lifecycle.init.sub1") {
        // sub-init1
      }
      trace.traceLocal("example.lifecycle.init.sub2") {
        // sub-init2
      }
    }
    trace.traceLocal("example.lifecycle.process") {
      // do the work
    }
    trace.traceLocal("example.lifecycle.end") {
      // end and clean-up
    }

    response.ok("complete")
  }
}
```

This will create multiple "Local Spans", one for each `traceLocal` invocation. These spans
will automatically calculate the execution time within each `traceLocal` closure - and
allows for nested closures. Most UI's that allow you to introspect trace data will surface
the local span information in a much more easy to digest way, because they are treated as
first class spans when visualizing the waterfall/gantt flow of a request, instead of
hidden annotation tags.

### Define the Server

```scala

class ExampleHttpServer extends HttpServer {

  // configure our router with our controllers
  override protected def configureHttp(router: HttpRouter): Unit = {
    router
      .add[ExampleController]
      .add[LifecycleController]
  }
}
```

In Finatra you add Controllers to the HttpRouter for your server. There are
more powerful things that you can do and configure about the server, but those are
beyond the scope of this discussion.

### Testing via EmbeddedHttpServer

```scala

class ExampleHttpServerFeatureTest extends FeatureTest {

  override val server =
    new EmbeddedHttpServer(new ExampleHttpServer)

  test("ExampleServer#scores correctly with 'name' query param specified") {
    server.httpGet(
      "/score?name=enbnt",
      andExpect = Ok,
      withJsonBody = """{"name": "enbnt", "score": 5}"""
    )

    // verify our trace annotation is present
    server.inMemoryTracer.binaryAnnotations("example.name", "enbnt")
    server.inMemoryTracer.binaryAnnotations.get(
      "example.multiplier"
    ) shouldBe None
  }
}
```

There are more tests in the example project, but for the sake of brevity, this
is where the magic happens! This code stands up our server, wrapped with the
`EmbeddedHttpServer` utilities, which will (lazily) bind the server under test to an
ephemeral port and initialize and connect a client, which is utilized under the
covers of the `server.httpGet` calls. We can access our trace annotation data via
the `server.inMemoryTracer` call, where we can assert that our annotation is
present and defined as expected (or not!). 

The `FeatureTest` trait manages the
lifecycle of our server under test, the auto-created clients, as well as the
in memory utiltiies. For example, the state of the `InMemoryTracer` and `InMemoryStatsReceiver`
will be cleared between test cases. The server and its borrowed clients will all be
cleaned up after all tests in the test class finish executing, preventing resource
leaks in JVMs that need to run MANY `FeatureTest`s. As with anything, there are some
gotchas/anti-patterns, but the framework is doing A LOT of heavy lifting and you get
to focus on verifying your application's behavior.

If you really want to learn more about Finatra, I recommend checking out the
[Finatra User's Guide](https://twitter.github.io/finatra/user-guide/).

*Aside: We'll possibly save this for another post, but part of the API design process
for the `InMemoryTracer` was in keeping things consistent and familiar to the
pre-existing `InMemoryStatsReceiver`. Consistency and familiarity are a HUGE part of 
API design and enabling developer velocity.*

### Testing via Zipkin UI

The `InMemoryTracer` also offers the ability to output the recorded spans
in various formats, including Zipkin's JSON format. I highlight Zipkin here,
as it has a tight integration with Finagle, but I am also unsure if the other
tracing tools support a `Upload JSON`-like utility that allows us to do what
we are attempting to do in this example.

```scala
  test("ExampleServer#processes a lifecycle request") {
    server.httpPost(
      "/lifecycle",
      postBody = "",
      andExpect = Ok,
      withBody = "complete"
    )

    server.inMemoryTracer.rpcs("example.lifecycle.init")
    server.inMemoryTracer.rpcs("example.lifecycle.init.sub1")
    server.inMemoryTracer.rpcs("example.lifecycle.init.sub2")
    server.inMemoryTracer.rpcs("example.lifecycle.process")
    server.inMemoryTracer.rpcs("example.lifecycle.end")

    // grab this output and save it to a file, which you can upload to Zipkin
    server.inMemoryTracer.print(InMemoryTracer.ZipkinJsonFormatter)
  }
```

The Zipkin UI contains a nifty feature which allows you to upload
a file containing a valid JSON payload with your Trace data to see
a rendering of it in the UI.

If you want to mess around, you can the sample Docker container via

```
$ docker run -d -p 9411:9411 openzipkin/zipkin
```

or you can do this by following along with any of the other instructions in the
[Zipkin Quickstart](https://zipkin.io/pages/quickstart.html) guide.

If you open `localhost:9411` in your browser, you can
click the `Upload JSON` button, which is located next to
the `Search by trace ID` box. Point to your file and you'll see
how you, your team, or your users will see your data visualized
in the tool. You can use this technique to highlight the most
critical parts of your application and make them stand out from
other standard annotations.

## Why?

Now for the backstory on why being able to verify trace data locally was
paramount. Finatra is an opinionated framework for building Finagle clients
and servers. Finagle exposes a lot of neat utilities, like automatic Tracing
integration. Finagle gives you things like automatic annotations for things that
hit the process boundary for an RPC (i.e. request latency, request path, request method)
and makes that Trace context available to people building their applications
and business logic &mdash; just like we did in this example.

***BUT,*** the mechanism that Finagle uses to propagate that Trace information to users
can be lost. Finagle automatically handles this, as long as you stay within the confines
of the framework and do things in a Finagle-y way (Finagle Clients, Finagle Servers, 
Finagle Threads, Twitter Futures, etc). If you try to integrate a Finagle thing with a 
non-Finagle thing &mdash; say a Java CompletableFuture &mdash; then the link can be broken. 
Your code will still compile and still run, but the magic glue isn't present and your 
annotation is lost.

***OH,*** did I forget to mention that when creating these tools, we ***ALSO*** found bugs in
how Finagle itself was generating Local Span data? Trace information was getting clobbered
with nested local spans. These utilities allowed us to verify and 
[fix](https://github.com/twitter/finagle/commit/77a7e774cd4f3bde8bbbfe67eb38732a7cc2ef3a)
the presentation of critical trace data.

The next question becomes "Where do I catch the fact that my annotations aren't present?"
If I find out my annotation is missing in production, maybe I can't debug a problem where I needed that information.
If I find out my annotation is missing in production, is it my code, the collector, the storage service, the query tool?
***How long will it take to narrow down the cause of your missing annotation?***
Wait, you verify your traces in a canary or staging environment and you caught it pretty quick?
Maybe that's actually OK if your feedback loop is tight. If other people are relying upon this
data for critical work or you have an engineering org that is scaled to hundreds/thousands of
engineers and it's minutes/hours/days/weeks before you can even know that something went wrong,
that can be valuable time or opportunity lost.

The ability to quickly verify, at the closest possible moment of the edit -> compile -> test loop
is one of the things that made Finatra so powerful. These checks can be dealt with as part of your
automated CI/CD pipeline. If you're going to fail, ***fail fast and fix it***.

I would love to see more frameworks and utilities offer the ability to verify the critical
observability signals that your application emits. The Finatra way is absolutely
not the only valid way, but with so much value being placed on the ability to introspect
observability data of your distributed systems, you probably want to invest in an automated way
to ensure that your data is there when you need it - or at least eliminate some variables, when
it isn't there.

I hope this post was useful, insightful, maybe even thought provoking or entertaining. If you
enjoyed this, please let me know. 

Until next time :wave: 
-Ian