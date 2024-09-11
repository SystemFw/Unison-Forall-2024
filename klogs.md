---
title: KLogs
author: Fabio Labella (SystemFw)
theme: solarized
highlightTheme: atom-one-light
revealOptions:
  transition: slide
  slideNumber: false
---

# KLogs

Distributed streaming on Unison Cloud

---
 
## About me

![](img/me.svg)



Notes:

I'm Fabio Labella, I'm the Principal Distributed Systems Engineer at
Unison Computing, so I spend most of my day thinking about Distributed
Systems design, and bringing it to life on Unison Cloud.


---

## Plan

- Designing _a_ streaming engine.
- Introduction to Unison Cloud.
- General techniques for distributed systems.

Notes:

So, plan for today. 
We will first look at the design of is a streaming engine, and I say
_a_ streaming engine because the design space is actually quite large,
depending on what use case your optimising your engine for, and then
on a myriad of architectural choices.
We'll then take a quick look at the main features of Unison Cloud, to
see which tools we have at our disposal to implement it.
And finally, we will dive into an actual distributed implementation,
which I hope will actually also serve as an introduction to several
general techniques and consideration for designing any distributed
system.
Let's get into it!

---

## KLogs

- &shy;<!-- .element: class="fragment" --> A streaming engine for OLTP. *(Not OLAP or ETL).*
- &shy;<!-- .element: class="fragment" --> *Roughly:* Kafka + a subset of Kafka-Streams.
- &shy;<!-- .element: class="fragment" --> Built entirely on Unison Cloud.
- &shy;<!-- .element: class="fragment" --> *Highly WIP and experimental*.

Notes:

KLogs is a streaming engine designed around OLTP, so transactional
processing at medium to high scale. Think about event driven logic
such as state machines or user workflows. It's not really intended for
analytical use cases where you want to run historical joins, or simple
ETL running at humongous scale, as we will see later when talking
about design tradeoffs.

It's roughly equivalent to the combination of a streaming platform
such as Kafka or Kinesis, plus a computation engine such as Kafka
Streams or Flink, although with more limited scope.

It's built in Unison and Unison Cloud from the ground up, but please
bear in mind that it is currently at prototype stage, there are still
missing features, unoptimised paths, etc.

----

### Api: KLog

```haskell
type KLog k v = ...
```

- &shy;<!-- .element: class="fragment" --> An infinite sequence of key-value pairs (messages).
- &shy;<!-- .element: class="fragment" --> Values for a given key are in FIFO order.
- &shy;<!-- .element: class="fragment" --> Values for different keys are independent and unordered.

----

### Api: pipelines

```unison [|1,2|1, 3-6|1,3,5-6|1,3-4|1, 7-11|1,7,10-11|1,7-9|1,12|]
ability Pipeline where
  merge : [KLog k v] -> KLog k v
  partition 
    : (k -> v -> Optional k2)
    -> KLog k v
    -> KLog k2 v
  loop
    :  s
    -> (s -> k -> v ->{Remote} (s, [v2]))
    -> KLog k v
    -> KLog k v2
  sink : (k -> v ->{Remote} ()) -> KLog k v -> ()
```


Notes:

Let's look at the Pipeline ability, which is how you write logic over
KLogs. It only has 4 ops: merge, partition, loop and sink.

Merge takes a bunch of KLogs, and merges them into one by emitting
messages as soon as they arrive.

Partition takes a KLog k v and transforms it into a KLog k2 v, i.e it
reroutes messages by changing their key. Remember that messages with
the same key will be in FIFO order, so partition essentially group
messages for later linear processing.

It does so with this function that computes a new key for a message,
and it returns Optional because you can also decide to simply filter
out the message by returning None.

Linear processing is done with the loop function, which is a stateful
transformation of the values of a KLog. Loop executes sequentially
over the values for a given key, and concurrently across different
keys.

It takes an initial per-key state, and a function that given a key
value pair and the old state, returns a new state, and publishes zero,
one, or more messages. You can look at it as a fold which acts per
key.

Finally, sink lets you perform side-effects at the end of a Pipeline,
for example to write messages to external storage. We will talk about
Remote later, but for now think about it as a version of the IO
ability that works on Unison Cloud.

----

### Api: KLogs

```unison [|1-2,6|1-2,7|1-2,8|]
ability KLogs where
  ...

KLogs.deploy : '{Pipeline} () ->{KLogs, Exception} ()
KLogs.named : Text ->{KLogs, Exception} KLog k v
KLogs.produce : k -> v -> KLog k v ->{KLogs, Exception} ()
```

Notes:

Ok, now we need an api to interact with our pipelines, which is the
KLogs ability, in a slightly simplified form: 
- deploy deploys a pipeline to start running it
- `named` creates or retrieves a named KLog to pass to our pipelines
- and produce lets us write messages to a KLog, so that we can
  interact with KLogs from the outside world

You might be wondering _where_ its the ability deploying pipelines or
storing klogs, but KLogs is an ability so we can abstract that choice
away: the appropriate handler will decide.

----o

### Example

```unison
example : '{KLogs, Exception} ()
example = do
  upper: KLog Text Nat
  upper = named "word-counts-uppercase"
  
  lower: KLog Text Nat
  lower = named "word-counts-lowercase"
  
  pipeline = do
    Pipeline.merge [upper, lower]
    |> Pipeline.partition (k _ -> [Text.toLowercase k])
    |> Pipeline.loop 0 cases _ counter value ->
         newCount = counter + value
         (newCount, [newCount])
    |> sink printCountJson

  KLogs.deploy pipeline

  upper |> KLogs.produce "F" 1
  -- {"type":"OUT","name":"f","count totals":1}  

  upper |> KLogs.produce "M" 3
  -- {"type":"OUT","name":"m","count totals":3}  

  lower |> KLogs.produce "f" 4
  -- {"type":"OUT","name":"f","count totals":5}  
  
  upper |> KLogs.produce "P" 2
  -- {"type":"OUT","name":"p","count totals":2}
```

Notes:

We have two key logs, both representing a stream of words with
respective counts from somewhere, however one KLogs is counting
uppercase words, the other lowecase, we want to do a case insensitive
count.

---

# End

Notes:

latest workaround: just add &shy;<!-- .element: class="fragment" -->
before your item content (when they contains links or anything non
text).

- &shy;<!-- .element: class="fragment" --> **This** is item one.
- &shy;<!-- .element: class="fragment" --> This is [item](https://...) *two*.
