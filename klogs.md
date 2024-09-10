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

I'm the Principal Distributed Systems Engineer at Unison Computing,
so I spend most of my day thinking about Distributed Systems design,
and bringing it to life on Unison Cloud.

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

## Test syntax
 
```unison
foo : Bar ->{Remote} Baz
foo = do
 a = readLine()
 printLine "world"
 
ability Yo where
  foo: Nat
  
type Foo = Bar Nat | Baz Text

bar = do
  Cloud.run.local Environment.default() do
```

Notes:

latest workaround: just add &shy;<!-- .element: class="fragment" -->
before your item content (when they contains links or anything non
text).

- &shy;<!-- .element: class="fragment" --> **This** is item one.
- &shy;<!-- .element: class="fragment" --> This is [item](https://...) *two*.
