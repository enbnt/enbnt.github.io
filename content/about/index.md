---
title: "About"
date: 2023-25-01T06:06:06Z
type: page
showTableOfContents: true
---

I am a Software Engineer located in the San Francisco Bay Area. I have years
of experience specializing in Distributed Systems, Performance Optimization, and
Open Source Software. I consider myself a strong developer advocate, focused on
improving the user experience from multiple angles, constantly seeking ways to reduce friction and make things better.

Here are some highlights from throughout my career. Consider this a condensed and
public form of a [brag document](https://jvns.ca/blog/brag-documents/).

***Jump To***

{{< toc >}}

## Twitter

![Twitter](images/twitter.png)

### Company Building

Throughout my various roles at Twitter, I activively participated in many company building
efforts, including:

  - Engineering Mentor/Mentee program, as a multiple time Mentor and Mentee
  - Numerous interview panels as both interviewer and shepherd across multiple teams/orgs/divisions
  - Technical Design Shepherd for multiple infrastructure projects
  - Platform Division Advisory/Steering committee member
  - Led the Platform Open Source Strategy advisory group
  - Company-wide Technical Radar (a guide for supported technologies) as both an author & reviewer 
  - Twitter University course instructor for multiple classes, such as "JVM Profiling" & "Dr. Finagle"
  - External Conference Speaker at both Monitorama 2017 and Monitorama 2018 [keynote]
  - Published Author in Hacker Noon (May 2017)

### Core API Platfrom - GraphQL [2021-2022]

![GraphQL](images/graphql.png)

The Core API team created a GraphQL platform that was tightly integrated with Twitter's
own internal serverless, virtual database - Strato. Product features exposed
through both the Public API and Twitter owned clients (i.e. iOS, Android, Reactive Web) were built
and exposed via Strato GraphQL bindings.

To give an idea for scale, at any point in time there were thousands [Operations](https://spec.graphql.org/June2018/#sec-Language.Operations) 
and tens of thousands of live [Documents](https://spec.graphql.org/June2018/#sec-Language.Document). 
There GraphQL runtime service processed **b**illions of [Fields](https://spec.graphql.org/June2018/#sec-Language.Fields) 
per second, across  hundreds of thousands of Queries per second, tens of **b**illions of queries 
per day.

For more public info on how GraphQL was used at Twitter, checkout

  - [GraphQL Is Not A Trap (2022) - Sasha Solomon](https://www.apollographql.com/events/virtual-event/graphql-summit-october-2022/graphql-is-not-a-trap/)
  - [200 Ok! Error Handling in GraphQL (2020) - Sasha Solomon](https://www.youtube.com/watch?v=RDNTP66oY2o)
  - [**twitter + graphql-java** - James Bellenger](https://github.com/graphql-java/graphql-java/discussions/2591)
  - [Twitter Blog: Rebuilding the Public API (2020)](https://blog.twitter.com/engineering/en_us/topics/infrastructure/2020/rebuild_twitter_public_api_2020) 
  - [Leverage va Autonomy in a Large Software System (2018) - Jake Donham](https://www.youtube.com/watch?v=0yX8wGCzWNs) (_Strato_)
  - [Strato: Twitter’s Virtual Database Powered by Microservices (2017) - Mike Solomon](https://www.youtube.com/watch?v=E1gDNHZr1NA) (_Strato_)

#### Runtime Trace Visibility Improvements

Twitter's GraphQL Platform is a multi-tenant platform that serves thousands of
[Operations](https://spec.graphql.org/June2018/#sec-Language.Operations), 
each of which corresponds to a different Twitter product surface 
(ex: ProfileTimeline, DM, HomeTimeline). Each GraphQL Operation is represented by 
multiple [Document](https://spec.graphql.org/June2018/#sec-Language.Document) states,
which correspond to a specific client release. The Operation and Document dictate 
different runtime performance characteristics of a GraphQL query’s execution.

In order to improve visibility/observability of runtime performance characteristics,
I led an effort to expose query execution lifecycle information in a way that could
be easily consumed by client engineers and backend service owners. The impact of this
work was removing the requirement to do days of manual collection and analysis by
domain experts to get an approximate cause of the issue, to instantly, automatically 
surfacing the cause to any user of the platform.

The work in this effort required fixing integrations between *[graphql-java](https://www.graphql-java.com)* 
and *[Finagle](https://twitter.github.io/finagle/)*, due to differing asynchronous 
execution and threading models. New integrated Trace verification utilities were 
added to *[Finatra](https://twitter.github.io/finatra/)*, which ensured that the
Continuous Integration (CI) suite would automatically detect any regression or
unexpected changes in behavior of the various framework integrations.

#### Home Timeline Migration

The HomeTimeline is the primary entrypoint for the Twitter user-experience. The team
had undertaken a multi-year effort to transition different Timeline endpoints 
from REST to GraphQL, the most difficult and unique being HomeTimeline. I acted as a
primary customer contact in the final year of the HomeTimeline GraphQL rollout, where 
our primary goal was to support a smooth migration and ensure the complexity and 
unique scale of the HomeTimeline operation caused minimal impact to Twitter users.

We successfully supported the migration of both iOS, Android, and Reactive Web 
clients to GraphQL. We accomplished this by implementing various performance 
improvements, which resulted in needing 50% fewer GraphQL Compute and Observability 
resources than initially projected, resulting in millions of dollars in annual 
savings.

#### Developer Experience / Stabilization

The Core API GraphQL platform included extensive Continuous Integration support, from
verifying that a valid GraphQL schema could be compiled and generated from internal data
bindings and logic (via Strato Catalog) to hot deploying changes to production services
taking live traffic. Hundreds of engineers made active contributions to this platform on
a daily basis, which results in significant hardware resource utilization before a change
makes it into the production system.

I led an effort within the team to simplify, streamline, and speed up our platform's build,
test, and runtime infrastructure. This effort included

  - Analysis and reducuction of build dependnecies to reduce build times over 10x (hours)
  - Utilize best practices for internal feature tests to reduce test runtimes over 10x (minutes)
  - Reduce static and dynamic runtime memory requirements by roughly 20x (gigabytes)

These changes yielded a reduction of hours of per-change validation, which correlated to
quarters to years of engineering time and resources saved. More importantly, the consumers
of the platform dealt with less friction in getting their features into production.

### Core Systems Libraries (CSL) [2018-2021]

![Finagle](images/finagle.png) ![Finatra](images/finatra.png)

The Core Systems Libraries team was responsible for developing, supporting, and
maintaining the foundational Twitter-stack libraries for distributed systems. These
libraries were consumed and extended by thousands of internal Twitter microservices,
as well as numerous Open Source consumers. The Open Source Twitter-stack projects owned by the
CSL team include:

  - [Finagle](http://www.github.com/twitter/finagle)
  - [Finatra](http://www.github.com/twitter/finatra)
  - [TwitterServer](http://www.github.com/twitter/twitter-server)
  - [Util](http://www.github.com/twitter/util)
  - [Scrooge](http://www.github.com/twitter/scrooge)
  - [Dodo](http://www.github.com/twitter/dodo)

#### Privacy Data Protections (PDP) / GDPR

I was responsible or leading the effort to integrate mTLS and service-to-service authorization
primitives to the Finatra framework. Finatra was highly leveraged at Twitter, across thousands
of production services, due to 

  - Reduced boilerplate when creating Finagle-based services
  - Common patterns for building services across multiple protocols
  - Ease of testing complex systems via Dependency Injection support
  - Value of tests by supporting verification via end-to-end repeatable Feature Tests
  - Example project scaffolding for generating new services within Twitter's infrastucture

I designed, implemented, documented, and supported the mTLS framework integration across all
supported protocols at Twitter for both clients and servers. This work made it nearly painless
for hundreds of engineers across thousands of services to make changes that would ensure that
Twitter's user data and systems were better protected. I am proud that my work helped support
both GDPR and FTC Consent Decree compliance, while retaining support for Finatra's motivating
feature set, with minimal burden/friction for developers.

I also continued to guide multiple teams on integrations/framework extensions that were not owned
by the Core Systems Libraries team, ensuring that these extensions would remain compliant and be
done in a consistent manner to reduce developer overhead/friction.

#### Developer Experience

The ***Finatra***: *Test Fast and Test Furious* effort was a small effort to reduce
the overall *edit -> compile -> test* loop latency for Twitter Engineers. Thousands
of test targets will create and exercise an `EmbeddedTwitterServer`, which had required
a minimum 1 second startup poll. I introduced notication hooks into the Twitter-stack and
testing infrastructure to minimize time waiting on server availability before test execution
began. Given Twitter's use of a monorepo and extensive CI verification of code changes, this
effort resulted in months of saved engineering time and resources, along with a typical startup
latency reduction of 99.5% - this was vital, given a pandemic caused lack of 
hardware availability and an urgent capacity crunch.

#### HTTP/2 Client Backpressure Support

The integration of HTTP/2 support within Finagle was a large multi-year project, 
which started years prior to my involvement. The last hurdle preventing wide adoption
was to enable support for Client Backpressure (a.k.a. Flow Control) to Finagle HTTP/2
clients. I worked directly with customer teams that were impacted by the lack of backpressure
support who had previously needed to explicitly disable HTTP/2, close the feature gap, verify
performance impact, and gradually rollout changes to live production services.

Over 8x reduction in established connection count for large services using HTTP. This further
reduced tail latencies, given the company-wide adoption of mTLS connections for service-to-service
communications, as encrypted connections require more overhead to establish than non-encrypted ones.

### Observability [2016-2018]

Twitter’s Observability stack is one of the most highly used and crucial pieces of infrastructure 
used to determine service health, along with historical and active incident investigation and 
resolution. The services are responsible for collection, storage, querying, visualization, and 
alerting of metrics data adopted by not only thousands of services, but compute infrastructure and 
networking - handling over 4.3 Billion unique metric writes per minute and millions of 
multi-dimensional queries per minute during my tenure.

Twitter's Observability stack is complex and world-scale. For more historical backround materials
see: 

  - [Observability at Twitter: Technical Overview [Part 1] (2016)](https://blog.twitter.com/engineering/en_us/a/2016/observability-at-twitter-technical-overview-part-i)
  - [Observability at Twitter: Technical Overview [Part 2] (2016)](https://blog.twitter.com/engineering/en_us/a/2016/observability-at-twitter-technical-overview-part-ii)
  - [Twitter Flight 2015 - Of the Order of Billions: Building Observability at Twitter by @caitie](https://www.youtube.com/watch?v=SC6XuD1tgcQ)
  - [Twitter Eng Blog: Observability at Twitter (2013)](https://blog.twitter.com/engineering/en_usa/2013/observability-at-twitter)

#### Monitorama 2018

Monitory Report: I Have Seen Your Observability Future. You Can Choose a Better One.

  {{<vimeo 274820724>}}

#### Monitorama 2017

Critical to Calm: Debugging Distributed Systems

  {{<vimeo 221060067>}}

#### Alerting System Migration

The migration away from Twitter's legacy alerting system to its next-gen alerting system started
in 2013. When I joined the team, the migration was roughly 80% complete, largely automated, but stuck due to missing features that were fundamentally incompatible with the prior system. I joined
and helped the team reach feature parity with the old system, design and implement new required
features, rearchited distributed scaling bottlenecks, refactored and made the overall observability
stack more efficient and reliable. If you watch my Monitorama 2018 talk, I discuss this migration 
in much more depth.

The impact of this work was a more stable and reliable alerting system for both customers and the
team owning/operating the services. The new system was more efficient with query execution resulting
in a nearly 50% reduction in queries executed per minute, which in turn dropped overall tail
latencies for queries by 5 seconds.

This migration was a multi-year effort, which required a large amount of work and empathy-building
with customers. It wasn't always easy, but we managed the difficult and rare feat of fully 
migrating 100% off of the legacy system, including complete shutdown of legacy services and code 
archival achieved.

#### Stabilization

The Observability Team was in flux. Team personnel was frequently changing, along with management
changes, reorgs, team mergers, and scope changes. I stepped in as Tech Lead from mid-2017 through
late-2018. I helped grow and educate the team from a few ICs to a dozen ready to contribute and
handle on-call responsibilities. We shipped new features, wrapped up a multi-year migration, 
took ownership of more services, grew the team 3x, and reduced the team's 24/7 pager load by
nearly 90%.

## IBM

### Watson AI [2011-2016]

![IBM Watson AI](images/ibm_watson.png)

#### Natural Language Classifier Service

The Natural Language Classifier service was IBM Watson's first attempt to break up the
monolothic Q&A (Jeopardy!) pipeline into distributed services made available via the
Watson Developer Cloud based offerings. I helped to optimize the Convolutional Neural Network's
algorithmic execution (> 2x speed up) and in memory data structures (700% memory reduction) by
creating a more efficient Linear Algebra/ML library. 

There was an aggressive timeline and tremendous team effort, as the service went from inception
to Public Beta in 3 months and was GA within 6 months of inception. I contributed by providing
presentations and social dissemination of best practices using Behavior Driven Developemnt,
Dependency Injection, and leveraging new CI build and test infrastructure. I also architected
and implemented the service's public REST API layer, including logging, metrics, monitoring,
authentication, PagerDuty integration, automated Integration Testing, and Performance & Accuracy
regression validation.

#### Watson Core Technology

The IBM Watson Core Technology group (not to be confused with the IBM Watson Research Group)
was formed as a result of the IBM Watson AI's performance on the "Jeopardy!" quiz show.

  {{<youtube lI-M7O_bRNg>}}

I was selected as one of the first 30 members to join this group and take the Jeopardy! based
Question and Answer (Q&A) system and turn it into both a product and business that scales.
The IBM Watson AI division was later formed from this group and become the fastest growing
division in IBM's history to $1 Billion. I was the engineer leading the memory reduction
initiative, where over a 50% savings was realized.

  > IBM’s researchers have shrunk Watson from the size of a master bedroom to a pizza­box­sized
  > server that can fit in any data center. And they improved its processing speed by 240%.

&mdash; <cite>[IBM's Watson Gets Its First Piece Of Business In Healthcare. (2013, February 8) Forbes](https://www.forbes.com/sites/bruceupbin/2013/02/08/ibms-watson-gets-its-first-piece-of-business-in-healthcare/?sh=5f545e155402)</cite>

I was often requested directly by customers for support, performance analysis, and implementation/
delivery of fixes. I helped customers achieve an 11x improvement in query latency on a 4x larger
corpus of data - on the same hardware. My performance automation, frameworks, and tools, which
included automated reporting and analysis were extended and leveraged by multiple customer teams.

### LotusLive Meetings [2008-2011]

![LotusLive](images/lotuslive.png)

Performance analysis and optimization for a globally distributed live meeting service.
Responsible for design and implementation of network level load generation tooling and
user modeling for features including slide share, screen share, remote control, 
audio/video, recording, polling, and text chat for thousands of simultaneous meeting attendees.

Included remote work opportunities with colleagues and customers in the U.S., Ireland,
Brazil, India, China, and Japan.
