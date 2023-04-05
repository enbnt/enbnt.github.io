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
be more efficient than the solution we outlined. Let's dive in! :diving_mask:

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
  - The data does not expect all intervals to be present in the series
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

Audio data

Fun fact :exclamation: I am only now realizing while writing this post, as I was checking for other references and usages, but Grafana uses this method!
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

### Service (Network)

### Storage (Disk)