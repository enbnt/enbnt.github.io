---
title: "Distributed Tracing and Testing"
date: 2023-02-02T13:16:35-08:00
draft: true
type: post
tags: ["observability", "tracing", "distributed tracing", "testing", "software development", "distributed systems"]
---

Hello again! With today's post we are going to talk about using
Distributed Tracing and Testing. For this post we will do a quick background
on the concept of Distributed Tracing and use the [Finatra](https://github.com/twitter/finatra)
Open Source project as a way to discuss some of these concepts, but also show
some concrete examples. The example code can be found in my [GitHub examples repo](https://github.com/enbnt/examples.enbnt.dev).


### Note

I spent years making contributions to the Finatra project. Following Twitter's
acquisition, I cannot comment as to the continued support or usage of Finatra
or any of Twitter's other OSS projects. I think the content and discussion are
still valid/relevant, as we can discuss things like design trade-offs or how
this work relates to or can inspire other works.

# Background

If you are unfamiliar with the concept of Distributed Tracing, 