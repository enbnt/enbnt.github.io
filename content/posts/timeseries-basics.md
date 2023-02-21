---
title: "Time Series - From A Coding Interview Lens"
date: 2023-02-21T05:05:05-08:00
type: post
showTableOfContents: true
tags: ["observability", "software development", "distributed systems", "interviewing", "time series"]
header:
  image: "avatar.png"
  caption: "Basic thoughts on Time Series and how they related to data structures
            and algorithms, through the lens of a coding interview."
---

***Jump To***

{{< toc >}}

## Intro

As I am personally ramping up on coding interview prep work, I thought it would
be a good time to share some insights on what sort of things I would look for
when interviewing potential candidates. Being a bit rusty in the interview game,
I am also using this as a device to rebuild the mental muscles of breaking down, 
simplifying, and communicating a problem. The toy problems we are given in an
interview setting are seldom like the job you're trying to get, but I think it's
important to be able to communicate problems and solutions with new potential
coworkers. Ideally, it gives both the candidate and interviewer a chance to gather
signals on what it might be like to work with each other.

For this post, I am going to use the concept of :chart_increasing:
[Time Series](https://en.wikipedia.org/wiki/Time_series) :chart_decreasing:.
While simple on the surface, time series are a vast and complex subject.
We are going to use a relatively simple example as a basis for discussion,
framed from the context of a coding interview setting. I haven't asked this 
question in years, but I think it makes for an interesting exploration.

*Disclaimer*: Before we dive in, I do want to highlight that I believe having
a strict and repeatable rubric and question flow is an important *part* of 
eliminating bias as part of evaluating a candidate. This discussion is **not** 
a single interview question or scenario! The expectations in a rubric should
be adjusted for the level of the candidate you're looking to hire - you should
not have the same expectations for an intern and a senior level candidate.
Please do **not** torture any candidates by trying to cover everything we 
will go over in this (and subsequent) posts in a single session.
It is not a good look :eyes:.

## :headphones: Musical Inspiration :musical_note:

{{< apple_music album="guidance/1116288956" >}}

{{< apple_music album="the-distance-between-zero-and-one/1059188690" >}}

## Basics

To set some background, we want to define a time series. For our discussion,
we will state that a time series is a sequence of tuples (or pairs) that 
have a `time` and `value` component. For our purposes, our
time series value component will be represented by a scalar. 
To illustrate:

```
  (t0, v0), (t1, v1), ..., (tn, vn)
```

Where the `t` are a timestamp and the `v` are a scalar value. 

Here is a more concrete example, where we have a 15 minute
interval between each data point, the values are integers,
and the series consists of 5 data points:

```
  (12:00, 5), (12:15, 10), (12:30, 8), (12:45, 9), (13:00, 7)
```

It might be easier to grasp by visualizing a line chart that
represents this data:

{{< img-svg "time-series-example" >}}

## Algorithms & Data Structures

### The Problem

Let's start off with a basic problem, which has practical applications
when you're working on a large scale observability system (or really anything
that uses time series data - stock tickers, machine learning, etc). We extend 
the Time Series concept slightly by mentioning that we have a Time Series Store, 
which allows us to retrieve a Time Series by an associated label - a `String`.

From this store, we have a method that allows us to retrieve and filter
values from the store based on a time range.

```scala
class TimeSeries {
    // ...
    def range(startTime: Long, endTime: Long): TimeSeries = ???
}

class TimeSeriesStore {

    def fetch(key: String, startTime: Long, endTime: Long): TimeSeries = ???

}
```

We can state some assumptions:
  - All data will exist locally in memory
  - The time series data will always contain timestamps that are ascending
  - The time series data will not change when it is being queried

### :stop_sign:Stop Here:stop_sign:

If you want to think about or solve the problem before we dive in, take some time
and come back to this section when you are ready. Otherwise, continue :vertical_traffic_light:.

### Thinking It Through

We have provided a decent skeleton that should hopefully have some questions start to
fire in the candidate's mind. Let's assume we're a candidate for a bit.

  * We are defining a `TimeSeries` class and returning that as a result of fetch
  * What does the `TimeSeries` class look like?
  * Our fetch method uses `startTime` and `endTime` as `Long` - why? (answer: epoch millis)
  * Are `startTime` and/or `endTime` exclusive or inclusive when filtering? (answer: both inclusive)
  * What are edge cases that need to be accounted for?
  * Are there any properties or assumptions we are allowed to make?

It's important to think of boundaries and negative tests. For example,
how do we handle:
  * A key that doesn't exist in the store?
  * A `startTime` that's greater than the `endTime`?
  * A negative value for `startTime` or `endTime`?
  * A time range that doesn't exist within the TimeSeries?

It would be good to draw up both a happy path and some boundary examples to use when
debugging or walking through the problem.

After some contemplation, we decide that we'll want to use a Map structure
(such as a HashMap) to store the label -> time series mapping. We can start
to fill in a little more of our skeleton:

```scala
class TimeSeries {
    // ...
    def range(startTime: Long, endTime: Long): TimeSeries = ???
}

// We can assume the store is given to us
class TimeSeriesStore(store: Map[String, TimeSeries]) {
    
    def fetch(key: String, startTime: Long, endTime: Long): TimeSeries = {
        require(startTime >= 0, s"startTime must represent a positive value, but receieved '$startTime'")
        require(endTime >= 0, s"endTime must represent a positive value, but receieved '$endTime'")
        require(startTime <= endTime, s"startTime ($startTime) must be <= endTime ($endTime)")

        store.get(key) match {
            case Some(ts) => ts.range(start, end)
            case None => throw new MissingKeyException(s"Key '$key' was not found in TimeSeriesStore $this")
        }
    }

}
```

In this example we decide to throw a `MissingKeyException` if the label
doesn't exist in the `TimeSeriesStore`. If you are a candidate, it's probably
worth discussing with an interviewer if it would be acceptable to change the
method signature to `Option[TimeSeries]` in order to avoid throwing a runtime
exception at this moment. You might also return an empty time series or a 
sentinel value, but this has an implication on the overall design of the 
problem. I want to save this discussion for a follow-up post, as this would
shift the focus from an algorithmic problem to a design problem. While it is
always worth considering design, the unsatisfying answer is that delving
deeper into design trade-offs is beyond the scope of this particular post
&mdash; stay tuned.

Maybe some bonus points for mentioning the possibility of using a 
[Trie](https://en.wikipedia.org/wiki/Trie) instead of a 
[HashMap](https://en.wikipedia.org/wiki/Hash_table), and if there was 
time we could discuss the trade-offs in the context of this solution. 
Depending on context, nearly any solution *could* 
work for looking up the `TimeSeries` mapping. It's not the critical part of 
the problem and we can get into more of the theoretical details when it 
comes to the `TimeSeries` implementation.

### A Naive Approach

```scala
class TimeSeries(data: Map[Long, Double]) {
  override def range(startTime: Long, endTime: Long): TimeSeries = {
    val filteredData = data.filterKeys { time =>
      time >= startTime && time <= endTime
    }
    new TimeSeries(filteredData)
}
```

This is a simple and naive approach, which uses a `Map` to represent the time series data.
It's fine if this is the first thing that jumps into your head, especially if you haven't
thought deeply about the characteristics of the problem. This is an opportunity to discuss
pros and cons of a `Map`-based implementation. 

For the sake of discussion, we will say that the Map implementation is a mutable `HashMap` -
think `java.util.HashMap` or `scala.collections.mutable.HashMap` or anything else that
is backed by a Hash table. Map lookups are theoretically O(1) time complexity and storing
the map structure in memory is theoretically O(n) space complexity. However, if you look
at our code above, we are iterating and filtering over all keys. Our current solution is 
theoretically O(n) and practically uses more time and space than a more optimal solution.

We have also ignored the stated assumption that the time series data would be stored
in a manner where time is always ascending. Our `HashMap` solution doesn't retain ordering.
If we were to iterate over the time series data, there is no guarantee that it will 
retain its insertion order (though, you might occasionally get lucky). This could be 
an opportunity to dive into a candidate's understanding of a `HashMap` implementation
and some of the edge cases. I have seen quite a few candidates assume that `HashMap`
lookups are nearly always constant time in practice and be surprised when I mention 
that I have seen multiple instances of production code, which have 

  - Worse runtime performance due to excessive hash collisions.

    - The worst case is that every entry hashes to the same bucket

    - This leaves you with larger practical space requirements AND 
      you're stuck iterating over linked list nodes for everything you do.

  - Memory leaks or excessive allocations occur due to keys that use identity equality

    - Hash collisions can occur, but the keys will only be equal if they're the same instance,
      thus resulting in an ever expanding Map on insertion or never finding the entry you had 
      previously inserted, despite having the same data (identity equality is often done 
      intentionally by third-party libraries you might depend on).

    - This is especially dangerous with unbounded map or cache implementations

  - Runtime performance hits due to expensive key hashing

    - This is sneaky and can bite you if you use Scala Case Classes or Java Record classes, 
      which contain collections, as the hashcode is auto-generated for you and will hash
      over entire collections (these don't generally make the best keys, regardless).

  - Runtime performance hits on resize 

    - This involves allocating larger sized arrays, copying, re-bucketing, and re-building
      the entire Map from scratch when a capacity limit has been met.

    - Size hints can sometimes mitigate this, but you might not always have control


You might decide that a [TreeMap](https://www.baeldung.com/java-treemap) or 
[ListMap](https://www.baeldung.com/scala/listmap) can just drop in here in order to retain
ordering, but this doesn't improve our best case theoretical or practical requirements. 
We still suffer from the above mentioned issues.

You might also see something like

```scala
class DataPoint(time: Long, value: Double)

class TimeSeries(data: List[DataPoint]) {
  override def range(startTime: Long, endTime: Long): TimeSeries = {
    val filteredData = data.filter { case DataPoint(time, _) =>
      time >= startTime && time <= endTime
    }
    new TimeSeries(filteredData)
}
```

which is also theoretical O(n) time and space complexity, but in practice requires
less memory overhead than the Map based implementation. There's more that you could
dig into here, as well &mdash; what is the backing List implementation (i.e. Array, LinkedList)
and how do we assume `.filter` behaves? How would you find out how that method
actually behaves? Google search, code search, IDE, man pages, profiling(!?) &mdash; 
all fine answers. I personally wouldn't spend a lot of time here, but we're
probably helping to hone our candidate's solution if we can ask the right questions.
This is still missing the detail that our data is given to us in ascending order
by time.

### A More Optimal Solution
```scala
class DataPoint(time: Long, value: Double)

class TimeSeries(data: Array[DataPoint]) {
  override def range(startTime: Long, endTime: Long): TimeSeries = {
    // assume we have the ability to run a binary search based on the
    // start time
    val startIdx: Int = binarySearch(data, startTime)

    // assume we have the ability to run a binary search based on the
    // end time - might be worth probing or walking through some
    // potential edge cases here
    val endIdx: Int = binarySearch(data, endTime) + 1

    val filteredData = data.slice(startIdx, endIdx)
    new TimeSeries(filteredData)
}
```

Other acceptable ways you might see this:

```scala
class DataPoint(time: Long, value: Double)

class TimeSeries(data: Array[DataPoint]) {
  override def range(startTime: Long, endTime: Long): TimeSeries = {
    // assume we have the ability to run a binary search based on the
    // start time
    val startIdx: Int = binarySearch(data, startTime)
    val filteredData = data.drop(startIdx).takeWhile(_.time <= endTime)
    new TimeSeries(filteredData)
}
```

```scala
class TimeSeries(times: Array[Long], values: Array[Double]) {
  override def range(startTime: Long, endTime: Long): TimeSeries = {
    // assume we have the ability to run a binary search based on the
    // start time
    val startIdx: Int = binarySearch(times, startTime)
    val endIdx: Int = binarySearch(times, endTime) + 1

    new TimeSeries(
      times = Arrays.copy(times, startIdx, endIdx), 
      values = Arrays.copy(values, startIdx, endIdx)
    )
}
```

Please, please, please don't implement binary search. Maybe talk through
how binary search improves the problem. If you can discuss or illustrate
an example of binary search, you can code up the algorithm or find a 
working version in a million different places. I don't see the value in
doing a memorization test on how to implement binary search. You can talk
through how finding each of the indices is O(log n), the array copy is 
still O(n). As a result, we are theoretically still O(n), but in practice, 
we have a much more consistent execution of our algorithm while using 
fewer compute and memory resources.

Another area to probe would be if we swapped an `Array` for another
abstraction, such as a `List` or `Iterable`. `Iterable` might not be
appropriate at all with a binary search, where-as `List` is wholly
dependent on the underlying implementation. If it's a `LinkedList`
or other non-indexed collection, the binary search would actually
result in worse performance than a more naive linear filter approach.

It's also probably fair to question if it's worth using binary search at
all, as the solution isn't as elegant or easy to read as one of the more
naive approaches (with the nittiest of nits). We sort of glossed over
scale requirements in this problem - in reality we're probably only returning
hundreds or thousands of data points in a range. If this is a command-line
utility that's used infrequently, you might never notice a difference.
If this is part of a distributed system, where the function is called
concurrently thousands of times per second, then this small optimization
could be a golden opportunity. 

In a high scale environment, you may find that the array copies 
are too much and instead use a shared immutable view, a ByteBuffer-like
approach, or a stream. Maybe there are ***other*** properties of time
series data that you can exploit that could make this even more efficient
and save or earn you even more money. We can go into some of those things
in a follow-up post, where I would like to discuss how to evolve this basic
problem or look at it from different angles.

***BTW***, I have seen the `Map[Long, Double]` implementation in production.
It was part of a critical system and it likely would have saved hundreds
of thousands to hundreds of millions of dollars by refactoring to the
kind of solution we discussed today. There is always an opportunity cost
trade-off and a risk factor depending on the level of complexity of the
refactor. Do you have or need to build a safety net to verify the changes
and what methods do you have at your disposal to roll out such a large
sweeping change? But, keep this info in your back pocket in case a similar 
opportunity arises after you've weighed your risks. Consider this a freebie.

## Closing Thoughts

While I did receive my Bachelors Degree in Computer Science, my
coursework didn't have a super strong focus on the things that I mentioned
in this post. I do remember having to implement a working compression 
algorithm over Thanksgiving weekend in C++ and we discussed things like 
[Dijkstra's Algorithm](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm)
and [A* Search](https://en.wikipedia.org/wiki/A*_search_algorithm) (which I
have literally never seen practically implemented in a production graph search
system), but I don't remember spending a ton of time on these fundamental building
blocks. I do remember distinctly doing other higher level coursework, where a
friend who had recently graduated and started in industry showed me the magic
utility of HashMaps and I started using them wherever I possibly could without 
fully understanding them.

It wasn't until I was working in industry where someone had mentioned the term
"hash buckets", which sounded familiar, but I didn't fully grasp what they were
talking about. It was fuzzy, it was something I was using, but never had to 
understand deeply. You can Google and read about these things, but sometimes it
just doesn't stick. Eventually you'll hit a problem where you need to understand
not just the theoretical nature, but the practical applications for these things.

I think it's worth learning these concepts, but also trying to find a practical
application AND get practical experience if you want to work on things like
large-scale distributed systems. I really want to emphasize that
***learning never stops***, therefore it is ***never too late to learn***. 
You absolutely **can** learn on the job. I could go on a tangent about how a
lot of this is the fault of a financially very successful, large company that
a lot of other tiny companies decided to mimic in hopes of having a similar 
level of success, without realizing that these methods are still incredibly 
biased (or that holding a near-monopoly couldn't have been a large factor 
in their success)... but I also want to acknowledge the reality that not 
every role can accept that steep of a growth curve. It's not fair to anyone
to throw someone into a situation where there is no support structure to 
allow for growth and a safety net &mdash; no one acting in good faith should
want to ensure a no-win scenario when it comes to your career or their team.

We need to break barriers down and open up more opportunities. My hope is that,
maybe armed with some of the knowledge from this post, we can both be one
step closer.

Until next time :wave: 
-Ian
