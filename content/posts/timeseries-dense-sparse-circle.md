---
title: "Time Series - Density, Sparsity, and Circular Buffer-ity"
date: 2023-03-31T12:12:12-07:00
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
of a few potential offshoots of the [Time Series - From a Coding Interview Lens](/posts/timeseries-basics) post.
In that post, we discussed some of the basic properties of time series and used binary
search to do a 

## :headphones: Musical Inspiration :musical_note:

{{< apple_music album="chon/1647225877" >}}

## Dense v Sparse

Tensor or Matrix - not the movie with keanu, trench coats, and green runes raining down CRT monitors - 
but the Linear Algebraic concept used throughout Machine Learning (i.e. Tensorflow)

A time series can be represented as a matrix

```
[t0, t1, t2, t3, ..., tn]
[v0, v1, v2, v3, ..., vn]
```

## Circular Buffers (aka Ring Buffers)

Audio data

## Context Matters

### Library (Memory/CPU)

### Service (Network)

### Storage (Disk)