---
title: "My Example Projects Use Bazel - What is Wrong With Me?"
date: 2023-03-07T12:12:12-08:00
type: post
showTableOfContents: true
tags: ["monorepo", "software development", "distributed systems", "blog"]
header:
  image: "avatar.png"
  caption: "Why did I setup Bazel for my example projects repository?"
---

***Jump To***

{{< toc >}}

## Intro

Hello, again :wave:! I've been busy formulating some new content that is
taking a little longer to do at the level I want and they won't land
the same way if I don't. Instead of waiting for that, let's take a moment
to talk about the [Bazel](https://www.bazel.io) build tool. More specifically, 
how I ended up using Bazel for the [examples.enbnt.dev repository](https://www.github.com/enbnt/examples.enbnt.dev).
I promise you, this is more OR less interesting than your expectations
(definitely not *exactly*).

## :headphones: Musical Inspiration :musical_note:

{{< apple_music album="oracular-spectacular/264720008" >}}

## :facepalm:Ian &mdash; What is Wrong With You?:facepalm:

O.K. Let's all take a deep breath. There may be some triggering terms
in this post and one of them is **monorepo**. Thankfully, the purpose
of this post is not to confirm or condemn the use of monorepos (or is
it monorepi). The topic has and will likely continue to be debated. 
If you're really interested, here are a few posts by others to get you
started:

  - [Monorepo: Google, Meta, Twitter, Uber, Airbnb and you?](https://medium.com/geekculture/monorepo-google-meta-twitter-uber-airbnb-and-you-1723db84d301) &mdash; Carolina Ramirez
  - [Advantages of monorepos](https://danluu.com/monorepo/) &mdash; Dan Luu
  - [Monorepos: Please don’t!](https://medium.com/@mattklein123/monorepos-please-dont-e9a279be011b) &mdash; Matt Klein

That still doesn't answer how I ended up using Bazel for my example projects, so let's dive in.

It all started when I was looking to write my [Distributed Tracing and Testing](/posts/tracing-and-testing/)
post. As I was trying to get a new project setup from scratch, I used the 
[Finatra: Getting Started](https://twitter.github.io/finatra/index.html) guide
to help streamline the process. As it turns out, this guide and the surrounding
documentation have not been updating with the OSS project release process. 
The guide no longer has an accurate list of dependencies when instructing users
how to get started via [scala-sbt](https://www.scala-sbt.org) (the
standard build tool for Scala projects). I had tried troubleshooting by iterating
through possibly missing dependencies and syntax errors for ***HOURS***. The error
messages were not super helpful. 

I want to clarify that I'm not trying to attribute
blame specifically to `sbt`, as the project layout and test dependencies for the
Twitter-stack projects ([Finatra](https://www.github.com/twitter/finatra), 
[Finagle](https://www.github.com/twitter/finagle), 
[Twitter-Server](https://www.github.com/twitter/twitter-server), 
[Util](https://www.github.com/twitter/util), 
[Scrooge](https://www.github.com/twitter/scrooge), 
[Dodo](https://www.github.com/twitter/dodo)) really only relied upon `sbt` as a
nicety for Open Source contributors and Sonatype Maven releases 
(ex: [finatra-http-server](https://mvnrepository.com/artifact/com.twitter/finatra-http-server)).
It's no secret that [Pants](https://www.pantsbuild.org) `BUILD` and later 
[Bazel](https://www.bazel.io) `BUILD.bazel` files have existed in the OSS repositories for 
a long time. There's even a public 
[Twitter Pants to Bazel Migration](https://opensourcelive.withgoogle.com/events/bazelcon2020/watch?talk=day1-talk2) tech talk by Borja Lorente that you can watch, if you're interested.
But the projects were never really maintained in an `sbt`-first manner, despite that
being a primary interface for contributors from the OSS community.

I have a lot more experience troubleshooting issues with Twitter's internal Pants and
Bazel than I do with `sbt`. So, I thought to myself, "Hey, self! Ian, I think I can 
call you Ian, we're on a first name basis, right? Ian &mdash; we have used Bazel, 
we've helped with the Pants to Bazel migration, we ***KNOW*** how these dependencies 
work. We have even used `exports` to simplify per-package and per-project dependencies
and know how the more complex test target dependencies work."

***"We. Have. Got. This."***

_I was incredibly wrong._

I began to question my life choices. Bazel is probably overkill for this one project,
but it could be useful for other example projects I wanted to do for the site. 
Would this effort be worth it? Was anyone ever even
going to read the blog post? Ian &mdash; What is Wrong With You?

## The Pain

From the start, I had suspected that I wouldn't be able to directly use the 
`BUILD` or `BUILD.bazel` files that are in the OSS Twitter-stack projects. 
At a minimum, the cross-project dependencies assume a monorepo structure, 
where each top level project has its own repository (note: [Dodo](https://www.github.com/twitter/dodo)
essentially exists to bridge this behavior when working with `sbt` and the OSS
projects). What I had not realized is that the syntax for defining scala library,
binary, and test targets didn't match the "out of the box" [Bazel Scala](https://github.com/bazelbuild/rules_scala) experience.

Beyond this, the Twitter-stack projects support multiple Scala version
build targets. The latest releases, for example, support scala `2.12`
and `2.13` compile targets. From a Sonatype Maven point of view, these
are entirely different artifacts. From a monorepo point of view, the
scala target version can be controlled separately.

As a more concrete example, here is a [BUILD file from Finatra's inject-app project](https://github.com/twitter/finatra/blob/f8fa860f1ed955e0050fe01ff284820d89b7cfc1/inject/inject-app/src/main/scala/com/twitter/inject/app/BUILD):

```python
scala_library(
    sources = ["*.scala"],
    compiler_option_sets = ["fatal_warnings"],
    platform = "java8",
    provides = scala_artifact(
        org = "com.twitter",
        name = "inject-app-core",
        repo = artifactory,
    ),
    strict_deps = True,
    tags = ["bazel-compatible"],
    dependencies = [
        "3rdparty/jvm/com/google/inject:guice",
        "3rdparty/jvm/org/slf4j:slf4j-api",
        "finatra/inject/inject-app/src/main/java/com/twitter/inject/annotations",
        "finatra/inject/inject-app/src/main/scala/com/twitter/inject/app/console",
        "finatra/inject/inject-app/src/main/scala/com/twitter/inject/app/internal",
        "finatra/inject/inject-core/src/main/scala/com/twitter/inject",
        "util/util-app/src/main/scala",
        "util/util-slf4j-api/src/main/scala/com/twitter/util/logging",
        "util/util-slf4j-jul-bridge/src/main/scala/com/twitter/util/logging",
    ],
    exports = [
        "3rdparty/jvm/com/google/inject:guice",
        "3rdparty/jvm/org/slf4j:slf4j-api",
        "finatra/inject/inject-app/src/main/scala/com/twitter/inject/app/console",
        "finatra/inject/inject-app/src/main/scala/com/twitter/inject/app/internal",
        "finatra/inject/inject-core/src/main/scala/com/twitter/inject",
        "util/util-app/src/main/scala",
        "util/util-slf4j-api/src/main/scala/com/twitter/util/logging",
        "util/util-slf4j-jul-bridge/src/main/scala/com/twitter/util/logging",
    ],
)
```

And a quick breakdown:

  - `scala_library` &mdash; how we specify a library target which will use
     the Scala compiler
  - `sources` &mdash; which files relative to the package/directory to include in
     this target
  - `compiler_option_sets` &mdash; compiler flags, in this case `fatal_warnings` will
     result in failed compilation when unused imports are present (amongst others).
  - `platform` &mdash; specifies JDK 8 version as the compile target for this specific
     target. This is useful if say JDK 11 or JDK 17 were the defaults, but for some
     reason this target wants to stay old school (more likely runtime perf differences
     that need debugging/verification).
  - `provides` &mdash; optionally generate an artifact for the target that can be consumed
     outside of Bazel. In this example, a `com.twitter:inject-app-core` jar is put into an
     internal Artifactory for consumption (this isn't Sonatype or OSS build related).
  - `strict_deps` &mdash; enable strict_deps, which requires each target to explicitly list
     dependencies (with possible exceptions for exports, this behavior wasn't always consistent
     or clear between Pants and Bazel).
  - `tags` &mdash; possibly meaningful tags for the build system.
  - `dependencies` &mdash; the dependent targets required to compile this target
  - `exports` &mdash; the dependencies our target wants to make available to other targets without
     having to explicitly list them as a dependency. *<- This is what I was really hoping to be able to take advantage of.*

And yet, it does not resemble [what I ended up with](https://github.com/enbnt/examples.enbnt.dev/blob/e3fa7f1af81259b7dca94cc48274904bc4696a93/tracing-and-testing/src/main/scala/dev/enbnt/example/BUILD.bazel):

```python
load("@io_bazel_rules_scala//scala:scala.bzl", "scala_library")

scala_library(
    name = "example",
    srcs = glob(["*.scala"]),
    visibility = ["//visibility:public"],
    deps = [
        "//tracing-and-testing/src/main/scala/dev/enbnt/example/domain",
        "@maven//:com_google_inject_guice",
        "@maven//:com_twitter_finagle_base_http_2_13",
        "@maven//:com_twitter_finagle_core_2_13",
        "@maven//:com_twitter_finagle_http2_2_13",
        "@maven//:com_twitter_finagle_http_2_13",
        "@maven//:com_twitter_finagle_netty4_http_2_13",
        "@maven//:com_twitter_finagle_stats_2_13",
        "@maven//:com_twitter_finagle_stats_core_2_13",
        "@maven//:com_twitter_finatra_http_annotations_2_13",
        "@maven//:com_twitter_finatra_http_core_2_13",
        "@maven//:com_twitter_finatra_http_server_2_13",
        "@maven//:com_twitter_finatra_jackson_2_13",
        "@maven//:com_twitter_inject_app_2_13",
        "@maven//:com_twitter_inject_core_2_13",
        "@maven//:com_twitter_inject_logback_2_13",
        "@maven//:com_twitter_inject_ports_2_13",
        "@maven//:com_twitter_inject_server_2_13",
        "@maven//:com_twitter_inject_utils_2_13",
        "@maven//:com_twitter_twitter_server_2_13",
        "@maven//:com_twitter_util_app_2_13",
        "@maven//:com_twitter_util_app_lifecycle_2_13",
        "@maven//:com_twitter_util_core_2_13",
        "@maven//:com_twitter_util_lint_2_13",
        "@maven//:com_twitter_util_slf4j_api_2_13",
        "@maven//:com_twitter_util_slf4j_jul_bridge_2_13",
        "@maven//:com_twitter_util_stats_2_13",
        "@maven//:jakarta_validation_jakarta_validation_api",
        "@maven//:org_slf4j_slf4j_api",
    ],
)

```

A quick breakdown and comparison:

  - `load` &mdash; not every project in my repo is assumed to be a Scala project,
     so this statement specifies that I want to load up rules for defining a
     `scala_library`. Without further customization or this load statement, the
     build would fail.
  - `name` &mdash; the name of the target. This is required for building and
     testing. In our Finatra OSS repo BUILD, this name was automatically implied
     by the package/folder and not necessary, but that isn't the out of the box
     behavior for Bazel.
  - `srcs` &mdash; This is slightly different syntax, but is essentially `sources`
     from our above example. Another difference is that the `globs` function is
     required in order to do glob expansion, otherwise static file names need to
     be explicitly listed.
  - `visibility` &mdash; If you want to create a target that other targets can
    list as a dependency, you need to specify a public visibility. The Bazel
    workspace can be configured to do this by default, but out of the box you
    need to list this for each target.
  - `deps` &mdash; this is pretty much the same as `dependencies` from the Finatra
    OSS repo BUILD, but you'll notice a different syntax here. Anything that specifies
    a build target within our repo can be referenced by the root repository, such as
    `//tracing-and-testing/...`. Because we have dependencies outside of our repo and
    haven't done a bunch of `3rd_party` customization of build targets, we have to list
    out each of the build targets and specify that they are coming from Maven. The
    ` "@maven//:com_twitter_finagle_base_http_2_13",` target represents `com.twitter:finagle-base-http_2.13`
    (as can be seen at [Maven Central](https://mvnrepository.com/artifact/com.twitter/finagle-base-http_2.13/22.12.0)).

This was also following some customizations to my [Bazel Workspace file](https://github.com/enbnt/examples.enbnt.dev/blob/main/WORKSPACE#L139-L227).
Did I mention that I encountered a pretty huge issue just getting Finagle's
[Netty](https://www.netty.io) dependencies to work, as most JDK based build
tools properly resolve the cyclic dependencies used by Netty's native libraries,
but Bazel doesn't? [This](https://github.com/bazelbuild/rules_jvm_external/issues/704)
Bazel issue is still open with some pretty gnarly workarounds, but you can take a look
[HERE](https://github.com/enbnt/examples.enbnt.dev/blob/main/third_party/netty/deps_workaround.bzl), 
[HERE](https://github.com/enbnt/examples.enbnt.dev/blob/main/third_party/netty/BUILD.bazel), and in
the previously mentioned [Bazel Workspace file](https://github.com/enbnt/examples.enbnt.dev/blob/main/WORKSPACE#L139-L227) for how I managed to get this working in my repo.

## Silver Linings

Getting past the whole Netty native library issue, I now have a working Bazel 
repository. I had an easier time playing whack-a-mole to add in missing dependencies
with Bazel than `sbt`. I found the error messages more readable and it allowed me to
get something working, which I'm not sure I would have accomplished with `sbt` alone. In `sbt` it
was difficult to sort out whether or not I had made an error with a missing dependency
or the syntax required to properly scope build/runtime/test dependencies.
Bazel would complain loudly that a certain import was missing from the classpath
and I could backtrace which project I needed to add as a dependency to resolve
the issue - if this would have been the case with `sbt` I could have progressed
much faster. Now that I have something working via Bazel, it could be relatively trivial
to take the Bazel build and translate the solution back to an `sbt`-based solution,
if we wanted something a little more Scala native in the Scala examples.

With Bazel I was also able to setup some guard rails on dependencies that should *only* be
test dependencies and not allowed to be referenced by other targets. This allowed
me to successfully reference things like the [EmbeddedHttpServer test artifacts](https://github.com/enbnt/examples.enbnt.dev/blob/main/tracing-and-testing/src/test/scala/dev/enbnt/example/BUILD.bazel)
via a custom `@testJar` qualifier (ex: `@testJars//:com_twitter_finatra_http_server_2_13_tests`) that
behaves similarly to the `@maven` qualifier *AND* ensures that non-tests cannot pull
these test-only dependencies into compilation (and hopefully as a result, the runtime classpath).

Using the `1:1:1` rule (a single target per-directory, representing a single package) also
comes with benefits. More modular artifacts promote better caching of compilation and test
results. While this isn't a massive gain with the existing samples, I'm expecting this to
payoff more with some of the other things I have in the works. If you aren't familiar, the
modules are cached unless they have themselves changed or a stated dependency has changed.
As a result, you can re-use unchanged artifacts without needing to do a full re-compile.
This also extends to test targets, which can assume a cached result if no underlying
change to that target was detected. You pay a heafty cost on the first run, but iterative
changes can have a lower cost.

I also have a pretty simple CI pipeline attached to the repo via the example repo's
[Github Action](https://github.com/enbnt/examples.enbnt.dev/blob/main/.github/workflows/main.yml).
The action triggers on a pull request to the repo and (as of this writing) will
attempt to build, test, and lint via `scalafmt`. I can continue to iteratively
evolve both my Bazel workspace and the action to meet my needs. The action also
uses a Bazel build cache phase, which could maybe speed things up with more
active development or across build steps (I didn't test this, it's just a
recommended practice). I also don't need to repeat this process for every 
new example project I want to create, as I have a shared and consistent 
approach. I don't have or plan to have a massive monorepo and there is unlikely
to be a ton of cross-dependencies or work happening
in a project/target closer to the root with many leaves (one of the largest challenges
of working on the Twitter-stack projects). I don't have some of the big tech co 
hurdles or problems.

The biggest reason I decided to stick it out and get Bazel working for this repo
was that I want the flexibility of creating example projects ***in different languages***.
Maybe I want to be able to show a full-stack UI and backend, with a frontend done
in React and a Java backend API layer. It meant that I could achieve this goal without
having to point to a bunch of different branches for different examples, with completely
different build systems, and a CI that would grow in complexity as those projects evolve.

## Closing Thoughts

Am I saying that you should use Bazel? ***NO.*** I still don't know how portable this
solution is. I tried to structure my Workspace in a way that things might be imported
into other workspaces, so that you don't need to solve the same problems
(*apparently* this is possible with Bazel). But I have no idea if any of the customizations
that I needed to make would prevent that from working as expected. I am not claiming to be
a Bazel or build tool expert and I'm not sure you could pay me enough for this to be
***my*** day job &mdash; I am so extremely grateful that others find joy in this and
make it easier for others. I will happily accept tips, pointers, and PRs. I will
happily continue to evolve the examples repository. 

I really don't know that I achieved one of my hopes of this process - which was 
to get this working for the Twitter-stack projects, so that others could make use of
these projects when solving similar problems. I think the path to get there, beyond 
setting up custom Bazel 3rd_party targets to ease dependencies, would be to fork each
of the Twitter-stack projects (minus Dodo), put them in a monorepo, and get all of the 
per-package BUILD files working with off-the-shelf Bazel in a way that is portable for
other projects. Do I think this would be worth the effort? ***NO***. Probably even
***HELL NO***. There are a few fringe exceptions, but it seems like an
awfully large risk, especially given today's lack of clear ownership and direction for 
those projects. As I mentioned in the [Distributed Tracing and Testing](/posts/tracing-and-testing/)
post, there is a lot to learn or discuss from these projects, but I can make no 
recommendation on their continued use.

I still feel like this effort was a success and that the knowledge from the 
experience is transferable. Perhaps this post is food for thought in your own 
endeavors. If any of this helps you with your own projects, that's the best
I can hope for. I'm sure I'll have more thoughts on this in the future, but
that should do it for this particular post.

Until next time :wave: 
-Ian