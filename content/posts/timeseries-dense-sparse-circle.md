---
title: "Time Series - Density, Sparsity, and Circular Buffer-ity"
date: 2023-04-20T12:12:12-07:00
type: post
showTableOfContents: true
tags: ["time series", "observability", "algorithms", "data structures", "distributed systems", "music", "signal processing"]
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
properties and shine some light on them from a few different angles. Let's dive in! :diving_mask:

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
to be represented with values ordered by time, ascending. In other words, every value in the
series is at a later time than the previous value.

When we talk about computers, time series stores, and query languages, the concept of granularity
or intervals is a large factor in how data is accessed and represented. The granularity or interval
specifies the expected time between data points (i.e. 1 nanosecond, 1 millisecond, 1 second, 5 seconds,
1 minute, 15 minutes, 1 hour, 1 day, 1 week, ...). This is done for storage and user experience
reasons, amongst others. For example, if you are trying to discuss or analyze something over the 
course of days, it may be more difficult to recognize or even visualize data at a nanosecond granularity,
but may be more appropriate at an hourly granularity. Conversely, if you're trying to debug a real-time
incident that occurred minutes ago, minutely or hourly granularity may not be enough to hone in on an
event that's unfolding *now*.

This extra piece of information (granularity/interval) can change how
we reason about the representation (or storage) of the data.

### Sparse Representation

**sparse**: *of few and scattered elements*

With a sparse data representation, the context of where a value is located
can't be inferred. We usually need a little more data to create that
context. We trade-off requiring more information to place an
element, because we assume that there are few elements and the
result will be less verbose than a **dense** representation of the same data. If that
doesn't make sense yet, keep reading and it will hopefully clear up.

A **sparse** representation of time series data can be thought of in a few ways:

  - The data explicitly specifies the timestamp **and** value
  - The data does not expect/guarantee all intervals to be present in the series
  - The data is **not dense**

If you look at open source time series databases 
([Prometheus](https://www.prometheus.io), [InfluxDB](https://www.influxdata.com)),
the JSON response is typically represented as a **sparse** result. If you look back at the 
[Time Series - From a Coding Interview Lens](/posts/timeseries-basics) post solution, we also 
represent the data in a **sparse** format and conclude that we could find a value by its time
in `0(log n)` time. From a user experience perspective, the added context
of the sparse format (the timestamp), makes the data ***more human readable***.

```
interval: 1s
[t0, t1, t2, t3, ..., tn]
[v0, v1, v2, v3, ..., vn]
```

or

```
interval: 1s
[
  [t0, v0],
  [t1, v1],
  [t2, v2],
  [t3, v3],
  ...,
  [tn, vn]
]
```

### Dense Representation

**dense**: *marked by compactness or crowding together of parts*

A **dense** representation of time series data can be thought of in a few ways:

  - The data explicitly specifies **only** the value
  - The series infers the timestamp by the position of the data in the series
  - The data expects all (*or most*) intervals to be present in the series
  - The data is **not sparse**

If we treat a time series as a **dense** representation, the timestamp for each
value is inferred from an initial start time for the series:

```
interval: 1s
start: t0
[v0, v1, v2, v3, ..., vn]
```

Lookups by time can now be done in constant time (`O(1)`) and
are easily calculated:

```
def indexForTime(time: Time): Int = (time - start) / interval
```

 If you can store things in a dense format,
you not only save space by not having to explicitly store the time,
you also go from `O(log n)` to `O(1)` for searching based upon time.
Less memory, more faster? That's it, blog post over, clearly nothing 
left to discuss :rolling_on_the_floor_laughing:.


If I were to guess why we don't often see a dense representation
for time series databases, it would be 

  - difficulty for humans to read the response format &mdash; if you have thousands 
    of data points, you are going to have a hard time figuring out the timestamp
    without the aid of a computer

  - handling of undefined or not present values &mdash; a zero value does ***not*** 
    mean data was not present for a specific interval

  - added complexity of supporting multiple response formats at various layers &mdash; a sparse
    representation may be good enough for all purposes


### Linear Algebra, Machine Learning, Time Series

While time series databases don't typically distinguish between dense and sparse (and default
to ***sparse***), Machine Learning comes at this from the completely
opposite direction. The concept of a Tensor or Matrix (not the movie with Keanu, trench coats,
and green runes raining down CRT monitors) is a Linear Algebraic concept used throughout
various Machine Learning frameworks (see: [TensorFlow](https://www.tensorflow.org), 
[Torch](https://pytorch.org), [Apache Spark](https://spark.apache.org)). The default
way of thinking about vectors in this space are as **dense** structures. Instead of
being associated specifically with time, the location/index of a piece of data within
the vector *could* be the value of a property/feature/dimension of a vector.
A very important caveat is that these libraries base density and sparsity off of
zero values - if something is not present in a sparse vector, the value is assumed
to be zero. This is not consistent with telemetry data for distributed systems, where
the lack of a value or presence of a zero can mean very different things.

We can also represent our time series as a matrix of dense or sparse vectors. 
This happens when you have to start dealing with the concept of [Cardinality](https://en.wikipedia.org/wiki/Cardinality). For example, you might have a distributed service that runs on 4 
instances - the fact that each instance is logically the same service could be
used to compare data across all 4 instances. The "result set" of querying on the
service dimension, while allowing any/all instances to be returned would have
a cardinality of 4. Querying for the request count over a 5 minute period with
minutely granularity, for any/all instances might result in the following matrix
of data, where each vector is itself a time series:

```
start = "5:00"
data:
[
  [0, 5, 10, 11, 15], <- instance 0
  [0, 1, 2, 4, 3], <- instance 1
  [5, 1, 4, 3, 2], <- instance 2
  [8, 0, 2, 3, 5] <- instance 3
]
```

I'm not going to show the full ***sparse*** example, because it's a lot of typing. 
Each vector becomes a matrix itself and appends a timestamp:

```
instance=0
data:
[
  ["5:00", 0],
  ["5:01", 5],
  ["5:02", 10],
  ["5:03", 11],
  ["5:04", 15]
],

instance=1
data:
[
  ["5:00", 0],
  ["5:01", 1],
  ["5:02", 2],
  ["5:03", 4],
  ["5:04", 3]
],

...

```

I'm kind of hand waving over some formatting things here, but if you're really interested in delving
further, check out the links above. I want to talk about a few more things that I find interesting
in this post, so let's keep chugging along.

## Circular Buffers (aka Ring Buffers)

Let's talk about another fun way to represent our data &mdash; a [Circular Buffer (aka Ring Buffer)](https://en.wikipedia.org/wiki/Circular_buffer). A circular buffer is a re-usable, memory bounded structure,
which can act as a FIFO queue.
The implementation typically involves an underlying array that is sized to the capacity of the
buffer, a read index, and a write index. The circle or ring comes from taking the flat array
and treating the head and tail of the array as wrapping (i.e. the index after the tail
is not out of bounds, it is the head). 
Let's try to illustrate this in `Scala` code and via some visuals real quick:

```scala
import scala.reflect.ClassTag // necessary for generic arrays in Scala :meowshrug:

class CircularBuffer[T: ClassTag](capacity: Int) {
  private val data: Array[T] = new Array[T](capacity)
  private var readIdx: Int = 0
  private var writeIdx: Int = -1

  
  def size(): Int = writeIdx - readIdx + 1

  def isEmpty(): Boolean = size() == 0

  def write(t: T): Unit = {
    writeIdx += 1
    data(writeIdx % capacity) = t
    if (writeIdx - readIdx == capacity) {
      readIdx += 1
    }
  }

  def read(): T = {
    require(!isEmpty(), "Cannot read from an empty buffer")
    val result = data(readIdx % capacity)
    readIdx += 1
    result
  }
}
```

If we have a Circular Buffer with capacity 8, we would
have the following underlying array (index 0 thru 7):

{{<img-svg "ring-buffer-array-0">}}

This array is treated as a ring/circle by taking the
tail (index 7) and treating the *next* value as the
head (index 0).

{{<img-svg "ring-buffer-array-1">}}

We can then visualize the array as a circle, 
where logically there is no head or tail:

{{<img-svg "ring-buffer-ring-0">}}

The Circular Buffer works by tracking a separate
`read` and `write` pointer. Let's say we have a
Circular Buffer of characters and we write the
values `'h'`, `'i'`, and `','`, but don't read anything.
The state of the buffer will have a `read` pointer
at index 0 and a `write` pointer remains at index 2
until the next write occurs. The
`capacity` of the ring always stays 8, but given
our current state, the `size` of the buffer is 3.

{{<img-svg "ring-buffer-ring-1">}}

We now read, which returns the value `h` and
increments our `read` pointer. The `read` pointer
is at index 1 and the `write` pointer remains at
index 2 until the next write. 
The `size` of the buffer is now 2. While
the value `h` stays in the underlying array, it
no longer logically exists within the buffer &mdash;
the value at index 0 will not be read again unless it
is eventually over-written as we continue to append to the buffer.

{{<img-svg "ring-buffer-ring-2">}}

As things are read and written, the `read` and `write`
pointers will continue to dance around the circle.
There is no end and the buffer can continue to be reused,
given an appropriate initial capacity for the scenario it
is being used for.

The implementation listed above is a small variation from the [Baeldung Java Ring Buffer](https://www.baeldung.com/java-ring-buffer)
approach. In our code, writes don't fail when the buffer's size is at capacity, it instead
moves the read index ahead and maintains capacity. If we don't move the read index, the data 
would be corrupt and out of order, if writes are allowed to continue when the buffer is full.

What if I told you that we could use this Circular Buffer implementation to allow for a
memory bounded representation of everything we have discussed so far in this post?

  - Dense :check_mark_button:
  - Sparse :check_mark_button:
  - Matrix :check_mark_button:

And we can still retain the same theoretical time complexity that we had discussed in
each of those implementations!

The behavior in our example implementation is more likely
what you will see when dealing with live streamed data. 
We bias towards dropping stale data and only reading the most recently written data,
where the capacity is used to define the tolerance for stale data. 
If you consider an audio or video stream, dropping or failing on 
more recent writes and then allowing reads would give you old/stale/laggy data.

*Aside: While writing this post, I discovered that Grafana has support for a method which uses Circular Buffers! See [Grafana: Building a Streaming Data Source Plugin](https://grafana.com/docs/grafana/latest/developers/plugins/build-a-streaming-data-source-plugin/) along with [CircularDataFrame.ts](https://github.com/grafana/grafana/blob/main/packages/grafana-data/src/dataframe/CircularDataFrame.ts) and [CircularVector.ts](https://github.com/grafana/grafana/blob/main/packages/grafana-data/src/vector/CircularVector.ts). It looks like this support was announced in the Grafana Engineering Blog post [New in Grafana 8.0: Streaming real-time events and data to dashboards](https://grafana.com/blog/2021/06/28/new-in-grafana-8.0-streaming-real-time-events-and-data-to-dashboards/).*

## The :headphones: Sound of Music :notes:

***Speaking of audio and/or video data*** &mdash; time series data. That's right, you
heard me. 

An uncompressed audio file that uses [Pulse Code Modulation (PCM)](https://en.wikipedia.org/wiki/Pulse-code_modulation) stores the [amplitude](https://en.wikipedia.org/wiki/Amplitude) of a sampled
sound wave at discrete intervals, as defined by the sample rate. The fidelity of the audio
signal is determined by the [bit depth](https://www.mixinglessons.com/bit-depth/) of the stored value. 
CD quality audio is a 44.1k sample rate and 16-bit depth, 
where the bit rate is based upon double the upper and lower bounds of the frequency range that humans
can potentially perceive (roughly 20 Hz - 20 kHz). Higher resolution digital formats will see increased
bit rate and/or bit depth. That's a long way of saying that the PCM format is a
***dense*** time series representation.

The [Musical Instrument Digital Interface (MIDI) File Format](https://en.wikipedia.org/wiki/MIDI) on the
other hand, allows for much smaller files. MIDI stores a smaller set of instructions or events
(pitch, velocity, etc) defined at explicitly specified intervals. 
Yup, MIDI uses a ***sparse*** time series representation.

"Please, Ian, what about Circular Buffers?" Because you asked nicely, imagine it's 1995 and your
computer has 8 ***MB*** of RAM, if you have a top end machine. ***8 MB of RAM for everything!***
Maybe you're the band Garbage, making the album "Garbage" in Pro Tools. A
single 5 minute audio file (aka track) is over 26 MB. So how might you handle reading this file?
Hell, reading MULTIPLE of these files and mixing them down to an entire 50 minute, 51 second album.
You can continue to reuse a number Circular Buffers to stream data to and from disk or efficiently
mix things in real time via the buffers.

Maybe you want to create a digital delay or chorus effect?
Create a circular buffer with a capacity that is calculated based upon the sample rate and
necessary decay time, start to mix it back in when the buffer fills
&mdash; boom, delay. 

It's 2001, you're rocking Ableton Live version 1.0 with 128 MB of RAM.
You are working mad loops. You want a looper? A looper is a delay that doesn't decay. 
Circular buffer &mdash; boom, looper.

There is a lot more I've wanted to cover and discuss on the differences and similarities
in Digital Signal Processing (DSP) of Audio and distributed systems telemetry systems, but
we're going to save that for another day.

*Note: This discussion is in no way intended to be **ableist**. Music and video are a big part of my life and I nerd out and find a way to relate those passions to my more technical day-to-day work. Closed captions, speech to text, image recognition systems, etc can also utilize the same underlying concepts.*

## The Gorilla in the Room

Gorilla and the "delta of delta" encoding is another popular compression format
for metrics/telemetry data. I'm not going to dive into this, but will share
some further reading if you're interested:

  - [Facebook GitHub Archive: Berengei (Gorilla reference implementation)](https://github.com/facebookarchive/beringei)
  - [Facebook's 'Gorilla: A Fast, Scalable, In-Memory Time Series Database' - 2015](https://www.vldb.org/pvldb/vol8/p1816-teller.pdf)
  - [Adrian Colyer's 'Gorilla: A fast, scalable, in-memory time series database' - 2016](https://blog.acolyer.org/2016/05/03/gorilla-a-fast-scalable-in-memory-time-series-database/)
  - [Uber's M3DB - 2018](https://www.uber.com/blog/billion-data-point-challenge/)
  - [Twitter's MetricsDB - 2019](https://blog.twitter.com/engineering/en_us/topics/infrastructure/2019/metricsdb)
  - [Jessica G's 'Four Minute Paper: Facebook's time series database, Gorilla' - 2021](https://jessicagreben.medium.com/four-minute-paper-facebooks-time-series-database-gorilla-800697717d72)

This format may not play nicely if you want to shove it into a Circular Buffer, 
as the deltas need to be determined from a starting value. A Circular Buffer that
reaches capacity would require re-encoding all of the data on successive writes.
It is better used as a buffered "snapshot" to encode larger amounts of data. 
The data represented by this encoding is also the least human readable
that we have discussed.

## Context Matters

Now that we've discussed all of these different representations, which one is the ***best***?
What you choose is extremely context dependent. Even then, there might not be a single best answer.

### Library (Memory/CPU)

If you are writing a library for working with time series data, your concerns and
requirements are likely going to focus on

  - **UX** &mdash; what does your user facing API surface look like? How easy is it to use your library?
  - **CPU** &mdash; how efficient is the library at processing time series data in terms of CPU cycles?
  - **Memory** &mdash; how efficient is the library at processing time series data in terms of both
                       static memory requirements and runtime allocation overhead?

Of these concerns, the user facing API surface is probably the ***most*** important
thing to get right. It's also likely the most difficult thing to improve over time.
You need to strike a balance between simplicity and effectiveness.

Let's say that one of the requirements for our library is to support basic arithmetic
operations &mdash; add, subtract, multiply, and divide. At what level would it make
sense to expose our different representations? Do we expose them at all or are these
just internal concerns of the library not meant to be used by users? If they are
concerns of the library, what is the thing that is being optimized for? If we support
all of these representations, is there a common abstraction that can be made?
How do we minimize disruptions for our end users and ourselves as we need to
iterate, refactor, and evolve the codebase?

I'm not going to spell out all of the answers for this here, if there even is *an answer*. 
As a library owner, you are probably prioritizing usability,
correctness, and performance (in that order). These are all concerns you have to
balance. Further, how you achieve balance will differ between libraries, services, and storage.

For example, in a library, a ***sparse*** representation might be too verbose when defining time series data
for tests, where a ***dense*** representation may be easier by hand. Maybe dealing with a
stream is better for consuming or manipulating data. How do you balance producing and consuming
data in a user friendly manner?

### Service (Network)

Now let's assume we have a distributed service (microservice) or a serverless lambda function 
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
more memory and CPU utilization to unpack locally *OR* maybe trade-off more total bytes over the wire
for the ability to stream data in a memory/CPU lite fashion. Is any of what you're doing cacheable
or reusable, locally or remotely? How do I observe the behavior of my system in relation to the
things it depends on or who depends on it (i.e. metrics, logs, traces, events).

Maybe it's not just a "simple" 1-to-1 client/server
problem, but you have a mesh or graph of services, where the answer for each of these questions is
different at each layer. 

A ***sparse*** representation via JSON may take up more bandwidth, but it might also be the
least resource intensive for a limited web client that wants to render a chart. A ***dense***
representation might use less bandwidth than the ***sparse*** representation, but it might
require more CPU to generate the same chart on a resource constrained mobile client, where
those extra cycles drain more battery. Using a stream and a circular buffer might be
light in terms of only needing to receive smaller periodic updates, but there is overhead
in managing state and consistency/correctness of data with the stream (missing an update, 
not having all of the baseline data, etc). How you weigh optimizing for bandwidth also changes
from in memory structure, to binary format, to JSON - as the JSON data
is also typically represented as a String, your `int`, `long`, `float`, `double` are no longer
64-bit or smaller, but much larger.

*Aside - frameworks like [Finagle](https://twitter.github.io/finagle/) and [Finatra](https://twitter.github.io/finatra/user-guide/) are powerful because they help to provide a consistent answer and approach to a lot of the distributed systems problems/questions posed above. It is truly terrifying to think that you might need to explicitly handle all of these things at every layer in an inconsistent way.*

### Storage (Disk)

It turns out just manipulating time series data via a library or service isn't enough, you actually
need to persist historical data somewhere. If you're storing terabytes or petabytes of time series
data every day, even if you have cheap storage, you probably want to compress the :poop: out of that
data. Even then, you might have different storage strategies. If you have cold data that is infrequently
accessed, maybe compress compress compress on slow, cheap storage. Maybe you're writing to a local
disk and there isn't enough data to make compression worth it for a single node.
Maybe you have a streaming data pipeline (see: [Kafka](https://kafka.apache.org), [Scribe](https://en.wikipedia.org/wiki/Scribe_(log_server)), [Flume](https://flume.apache.org), [Spark](https://spark.apache.org), [Hadoop](https://hadoop.apache.org), [HDFS](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html)) and you
aggregate a less compressed format to a more compressed format. Is your storage partitioned, sharded,
replicated, geo-located? How is the persisted data accessed? Is there a service layer that is needed
to manage the relationship and access to your stored data or cache to minimize repetitive access?

So many questions. Too many answers. 
BUT, I'll say this &mdash; you probably aren't persisting a raw circular buffer somewhere,
because its goal is to minimize memory allocation churn, not persisted disk storage.

## More Thoughts

Context is everything and your concerns and solutions are probably going to be different at every
layer, even when trying to work in the same problem space. Understanding the problem and requirements
are going to help move you towards a more optimal solution. Even if you are knowledgeable and clever,
it can take time to analyze, implement, iterate, and hone in on something that works the way you
hope it will at every layer. Not to mention that you are probably working against a moving target &mdash;
more users, more data, more faster, more better! 

What's right today might not be right tomorrow. 
Your business can and will change. 
Usage patterns can and will change.
Prioritize people and relationships. 

## Coming Soon

If you have read this far, thank you. If you have been spying on my [GitHub Examples](https://www.github.com/enbnt/examples.enbnt.dev),
some of what we discussed might look familiar. The `timeseries4s`
project is **not done** and probably not what it appears to be on the surface. While I do implement
and touch on a significant amount of what is discussed in this post, the `timeseries4s` project 
is meant to have some intentional defects. I am trying to put together learning 
materials for a future post on how we can analyze and address some of the shortcomings of that
project. Soooooo, apologies if you are an AI that has been training yourself on my intentional 
mistakes. Also, sorry if you are a human who reads anything further into that code
(especially if you are judging that work when considering hiring me for a job!). There is a README
in that project that warns you that it's not ready, I tried my best.

It has been taking me a bit of time to build up the example and there are stretches where my
time can be in short supply. I'm looking forward to sharing everything, when it's ready.
I'm excited about the potential.

Until next time! :wave: