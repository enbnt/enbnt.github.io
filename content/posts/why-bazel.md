---
title: "My Example Projects Use Bazel - What is Wrong With Me?"
date: 2023-03-06T12:12:12-08:00
type: post
showTableOfContents: true
tags: ["monorepo", "software development", "distributed systems", "blog"]
header:
  image: "avatar.png"
  caption: "Why I setup Bazel for my example projects repository"
---

***Jump To***

{{< toc >}}

## Intro

Hello, again :wave:! I've been busy formulating some new content that is
taking a little longer to do at the level I want to and they won't land
the same way if I don't - so let's talk about the [Bazel](https://www.bazel.io)
build tool. More specifically, how did I end up using Bazel for the
[examples.enbnt.dev repostiory](https://www.github.com/enbnt/examples.enbnt.dev).
I promise you, this is more OR less interesting &mdash; BUT definitely not 
*exactly* &mdash; of what you were expecting from this topic.

## :headphones: Musical Inspiration :musical_note:

{{< apple_music album="oracular-spectacular/264720008" >}}

## :facepalm:Ian &mdash; What is Wrong With You?:facepalm:

O.K. Let's all take a deep breath. There may be some triggering terms
in this post and one of them is **monorepo**. Thankfully, the purpose
of this post is not to confirm or condemn the use of monorepos. The
topic has and will likely continue to be debated. If you're really
interested, here are a few posts by others to get you started:

  - [Monorepo: Google, Meta, Twitter, Uber, Airbnb and you?](https://medium.com/geekculture/monorepo-google-meta-twitter-uber-airbnb-and-you-1723db84d301) &mdash; Carolina Ramirez
  - [Advantages of monorepos](https://danluu.com/monorepo/) &mdash; Dan Luu
  - [Monorepos: Please don’t!](https://medium.com/@mattklein123/monorepos-please-dont-e9a279be011b) &mdash; Matt Klein

That still doesn't answer how I ended up using Bazel for my example projects.
It all started when I was looking to write my [Tracing and Testing](/posts/tracing-and-testing/).
The [Finatra: Getting Started](https://twitter.github.io/finatra/index.html) guide
is no longer updating with releases and the process for starting a new project with
the accurate list of dependencies via [scala-sbt](https://www.scala-sbt.org) (the
standard build tool for Scala projects). I had tried troubleshooting by iterating
through possibly missing dependencies and syntax errors for ***HOURS***. The error
messages were not super helpful. I want to clarify that I'm not trying to attribute
blame specifically to `sbt`, as the project layout and test dependencies for the
Twitter-stack projects ([Finatra](https://www.github.com/twitter/finatra), 
[Finagle](https://www.github.com/twitter/finagle), 
[Twitter-Server](https://www.github.com/twitter/twitter-server), 
[Util](https://www.github.com/twitter/util), 
[Scrooge](https://www.github.com/twitter/scrooge), 
[Dodo](https://www.github.com/twitter/dodo)) really only relied upon `sbt` as a
nicety for Open Source contributors and Sonatype Maven releases 
(ex: [finatra-http-server](https://mvnrepository.com/artifact/com.twitter/finatra-http-server)).

It's no secret, as the [Pants](https://www.pantsbuild.org) `BUILD` and later 
[Bazel](https://www.bazel.io) `BUILD.bazel` files have existed in the OSS repositories for 
a long time. There's even a public 
[Twitter Pants to Bazel Migration](https://opensourcelive.withgoogle.com/events/bazelcon2020/watch?talk=day1-talk2) tech talk by Borja Lorente that you can watch, if you're interested.
I have a lot more experience troubleshooting issues with Twitter's internal Pants and
Bazel than I do with `sbt`.

I thought to myself, "Hey, self! Ian, I think I can call you Ian, we're on a first name
basis, right? Ian &mdash; we have used Bazel, we've helped with the Pants to Bazel
migration, we ***KNOW*** how these dependencies work. We have even used `exports`
to simplify per-package and per-project dependencies and know how the more complex
test target dependencies work."

***"We. Have. Got. This."***

I was incredibly wrong. 

I began to question my life choices. Bazel is probably overkill for this one project,
but it could be useful for other example projects I wanted to do for the site. 
Would this effort be worth it? Was anyone ever even
going to read the blog post? Ian &mdash; What is Wrong With You?

## The Pain

I suspected that I wouldn't be able to directly use the `BUILD` or `BUILD.bazel` 
files that are in the OSS Twitter-stack projects. At a minimum, the project
cross-project dependencies assume a monorepo structure, where each top level
project has its own repository (note: [Dodo](https://www.github.com/twitter/dodo)
essentially exists to bridge this behavior when working with `sbt` and the OSS
projects). What I had not realized is that the syntax for defining scala library,
binary, and test targets didn't match the "out of the box" [Bazel Scala](https://github.com/bazelbuild/rules_scala) experience.

Beyond this, the Twitter-stack projects support multiple Scala version
build targets. The latest releases, for example, support scala `2.12`
and `2.13` compile targets. From a Sonatype Maven point of view, these
are entirely different artifacts. From a monorepo point of view, the
scala target version can be controlled separately.
