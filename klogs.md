---
title: KLogs
author: Fabio Labella (SystemFw)
theme: solarized
highlightTheme: solarized-light
revealOptions:
  transition: slide
  slideNumber: false
---

# KLogs

Distributed streaming on Unison Cloud

---
 
## About me


![](img/me.svg)


---

## Plan

```unison
foo : Bar ->{Remote} Baz
foo = do
 printLine "world"
 
ability Yo where
  foo: Nat
```
```scala
def foo = 1
```

```java
int a = 3;
```

Notes:

latest workaround: just add &shy;<!-- .element: class="fragment" -->
before your item content (when they contains links or anything non
text).

- &shy;<!-- .element: class="fragment" --> **This** is item one.
- &shy;<!-- .element: class="fragment" --> This is [item](https://...) *two*.
