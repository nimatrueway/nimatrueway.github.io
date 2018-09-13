---
layout:      blogpost
title:       Scala Quick Guide
img:         002-scala-guide-logo.png
date:        2017-09-05 00:00:00 +0000
tags:        ["Scala", "Guide", "Video"]
excerpt:     A small video series covering most concepts, methods and popular frameworks of Scala language
---

<center>
<img src="/img/posts/002-scala-guide-logo.png" />
</center>

## Description
A short guide on scala that **targets average developers** who intend to learn scala syntax and popular frameworks for **scalable web application development**. we quickly go over basic scala syntax, then explain how to use **modern features** of scala to build applications that are **much less error-prone**, and are able to easily **scale with your user-base growth**.

## Prerequisites
 - At least **6 months experience** in software development.
 - Knowledge of **procedural** and **object oriented** programming concepts
 - Familiarity with **scalability concepts** (e.g. Asynchronous Programming, Clustering, Load Balancing..)
 - Have **unix-based** OS and **IntelliJ** installed

## Syllabus

### [![youtube]({{ site.baseurl }}/img/youtube.png) &nbsp; Introduction](https://www.youtube.com/watch?v=1ycGtnqw5b0&list=PLWurvNkeUyp4HUocRQ8XTRQ61L0gZm5WT&index=1)
- Description
- Prerequisites
- Why Scala
- Scala Features

### [![youtube]({{ site.baseurl }}/img/youtube.png) &nbsp; Session #1](https://www.youtube.com/watch?v=cVj6HEeoBHY&list=PLWurvNkeUyp4HUocRQ8XTRQ61L0gZm5WT&t=1744s&index=2)
- Scala syntax (basic)
  - Create sbt project
  - Function, variable, field, expressions<br/>
    <small>(immutability, function-purity)</small>
  - Package, import
  - Implicits (1)<br/>
    <small>(introduction)</small>
  - Pattern matching (1)
  - Tuple
  - Syntactic sugars<br/>
    <small>(for-comprehensions, currying, tuple, function, infix-operator)</small>
  - Traversables (1) - Traversable, Iterable, Seq + sub-types<br/>
    <small>(Interconversion with Java collections)</small>
  - String Interpolation

#### Session #1 - Assignment <small><a href="https://gist.github.com/nimatrueway/aff9336eff532c64ea50c46b870b2bb0">[Solution]</a></small>

Write a scala application that does the same as unix `ls -l` command. Take a `path` as argument then list all files (plus directories) in it. In addition, print all these information for each file: permissions, owner user, owner group, size in bytes, Last modification date.

```
drwxr-xr-x  root   root     4096 Jul 20  2016 acpi
-rw-r--r--  root   root     3028 Jul 20  2016 adduser.conf
drwxr-xr-x  root   root     4096 Sep  1 14:48 apt
```

### [![youtube]({{ site.baseurl }}/img/youtube.png) &nbsp; Session #2](https://www.youtube.com/watch?v=9_mIEC8RzQM&index=3&list=PLWurvNkeUyp4HUocRQ8XTRQ61L0gZm5WT)
- Scala syntax (advanced)
  - Class, Object, Companion-object, and Trait in depth<br/>
    <small>(Modifiers, Getter-Setters, Nested elements, Mix-in)</small>
  - Self-type
  - Type argument/parameter<br/>
    <small>(Type bounds, Variance)</small>
  - Type-class pattern
  - Implicits (2) - in depths<br/>
    <small>(Implicit types, finding-paths)</small>
  - Type alias, Type member
  - Scala type-hierarchy and type-inference algorithm in depth<br/>
    <small>(Any, AnyVal, AnyRef, Unit, Null, Nothing, Nil)</small>
  - PartialFunction
  - Better alternatives for `null` and `try-catch`<br/>
    <small>(Option, Either, Try)</small>
  - Monads
  - Asynchronous programming tools<br/>
    <small>(Future, Promise, ExecutionContext, ForkJoinPool)</small>
  - Pattern matching (2) - Extractors
  - Case-class
  - Enumerations implementation strategies<br/>
    <small>(scala.Enumeration, Sub-classing)</small>
  - Traversables (2) - Set, Map, SQL-like querying on collections

#### Session #2 - Assignment

Write a program that non-blockingly reads and parses multiple svg files into meaningful well-structured hierarchy of objects using a single threaded ExecutionContext. Then use type-classes to add these capabilities to your svg-class :

1. Render it into a file with rasterized graphic format. (e.g. **png**)
2. Print its shapes in a readable text format.
<p>&nbsp;</p>
**Note 1**: Make sure your engine minimally supports these [sample svg files]({{ site.baseurl }}/files/svgsamples.zip), and you have covered all these tags/attributes:

  - svg [<small>viewBox</small>]
  - g [<small>stroke/stroke-width/fill/fill-rule</small>]
  - path [<small>d=<font color="#888">m/M/l/L/c/C/s/S/q/Q/a/A/v/V/h/H/z/Z</font></small>] [guide](https://css-tricks.com/svg-path-syntax-illustrated-guide/)
  - rect [<small>x/y/width/height</small>]

**Note 2**: Make use of scala features to achieve the cleanest, most legible code that everybody can comprehend in a short look. Keep your functions pure (no side effect, and no more than 15 lines in body)

**Note 3**: Use [Doodle (a recent Scala library)](https://github.com/underscoreio/doodle) or [SWT graphics library](http://www.eclipse.org/swt/snippets/) for drawing.

**Note 4**: Use java-nio to read a file non-blocking way. ([sample code](https://gist.github.com/tovbinm/f73849aff169d1ebeb97))

### Session #3
- Writing test with **ScalaTest** DSL
- Macros overview
- Scala Patterns
  - Algebraic Data Types
  - Type Classes (Adhoc Polymorphism)
  - Cake
  - Thin Cake
  - High Priority - Low Priority

### Session #4
- Writing web application server with Play!
  - Server-side page rendering using Twirl
  - Rest services and Json serializer/deserializer
  - Static file serving
  - Handling file uploads
  - Authentication using Silhouette
  - Writing custom filter

### Session #5
- Functional Relational Mapping with Slick/Quill
  - Implementing basic CRUD functionality
  - Writing custom type-to-column mapper
  - Writing user-defined functions
  - Mapping subclasses

### Session #6
- Using Akka Actor System
  - Basics
  - Writing micro-services
  - Setting protocol
  - Integration with Kafka
  - Clustering and load-balancing

### Session #7
- Meta programming
  - Using scala reflection
  - Using scala.meta

### Session #8
- **[** To be announced **]**

## Guide

[Y] [Youtube Playlist](https://www.youtube.com/playlist?list=PLWurvNkeUyp4HUocRQ8XTRQ61L0gZm5WT)

[T] [Telegram Channel](https://t.me/scalaguide)
