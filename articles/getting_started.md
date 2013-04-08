---
title: "Getting Started with Titanium, a Clojure graph library"
layout: article
---

This guide combines an overview of Titanium with a quick tutorial that
helps you to get started with it. It should take about 10 minutes to
read and study the provided code examples. This guide covers:

 * Features of Titanium
 * Clojure and Titan version requirements
 * How to add Titanium dependency to your project
 * A very brief introduction to graph databases and theory
 * Basic operations (creating vertices and edges, fetching vertices, graph queries)
 * Transaction control basics

This work is licensed under a <a rel="license"
href="http://creativecommons.org/licenses/by/3.0/">Creative Commons
Attribution 3.0 Unported License</a> (including images & stylesheets).
The source is available
[on Github](https://github.com/clojurewerkz/titanium.docs).


## What version of Titanium does this guide cover?

This guide covers Titanium 1.0.x.


## What Titanium is

Titanium is a Clojure graph library build on top of [Aurelius
Titan](http://thinkaurelius.github.com/titan/). It is easy
to use, strives to support every Titan feature, supports multiple
storage backends (Cassandra, HBase, BerkeleyDB Java Edition), takes
the "batteries included" approach and is well maintained.

Find more background information in

 * [The Rise of Big Graph Data](http://www.slideshare.net/slidarko/titan-the-rise-of-big-graph-data)
 * [Big Graph Data With Cassandra](http://www.youtube.com/watch?v=ZkAYA4Kd8JE)


### What Titanium is not

Titanium is not a database. It relies on other data store engines to
take care of on-disk durability. Titanium's value is in providing an
expressive Clojure API for graph operations.

Titanium is not an ORM/ODM. It does not provide graph visualization
features, although this is an interesting area to explore in the
future versions. Titanium may or may not be Web Scale and puts
correctness and productivity above sky high benchmarks.


## Supported Clojure versions

Titanium is built from the ground up for Clojure 1.4 and later. The most recent
stable release is always recommended.

## Adding Titanium Dependency To Your Project

### With Leiningen

    [clojurewerkz/titanium "1.0.0-alpha4"]

### With Maven

Add Clojars repository definition to your `pom.xml`:

{% gist 65642c4b53d26539e5f6 %}

And then the dependency:

    <dependency>
      <groupId>clojurewerkz</groupId>
      <artifactId>titanium</artifactId>
      <version>1.0.0-alpha4</version>
    </dependency>

It is recommended to stay up-to-date with new versions. New releases
and important changes are announced
[@ClojureWerkz](http://twitter.com/ClojureWerkz).


## Brief Introduction to Graph Databases

A graph is a data structure that represents objects and the
connections between them. Connected objects are called "vertices" or
"nodes" and connections are called "edges" or "relationships".
Vertices may have a multitude of different properties. A vertex could
be used to represent a person and so might have properties for age,
name, and gender. An edge links two vertices and it too can have many
different properties. If two vertices represented two people, an edge
connecting them could represent their friendship and have properties
that detailed the last time they spoke and when the friendship
started. In addition, there can be multiple edges between any two
given nodes and edges can have a direction associated with them. 

## Opening a Graph

Titanium is built on top of Titan which can be configured, for
example, to use different durable storage backends:

 * [BerkeleyDB Java Edition](http://www.oracle.com/technetwork/database/berkeleydb/overview/index-093405.html)
 * [Apache Cassandra](http://cassandra.apache.org)
 * [Apache HBase](http://hbase.apache.org)

as well as in-RAM storage (used primarily for automated testing and experimentation in the REPL).

In this guide, we will be introducing various features of Titanium
using an embedded instance of BerkeleyDB. For information about how to
use other database backends, please see the [configuration guide]().

To start working with Titan, we will pass a map of configuration
properties to `clojurewerkz.titanium.graph/open`.

``` clojure
(ns myservice.graphs
  (:require [clojurewerkz.titanium.graph :as tg]))

(defn- main
  [& args]
  ;; opens a BerkeleyDB-backed graph database in a temporary directory
  (tg/open (System/getProperty "java.io.tmpdir"))
  (comment Work with the graph here))
```

`clojurewerkz.titanium.graph/open` will use the Titan API to open up a
new graph and store the resulting object in a dynamic var.

## Creating Vertices

Please note that `clojurewerkz.titanium.graph/transact!` must always
be the context in which any code touching the database is run. For
more information, please see the [transaction management guide]().

Vertices are created using the
`clojurewerkz.titanium.vertices/create!` function.

``` clojure
(require '[clojurewerkz.titanium.graph :as tv])

(tg/transact! 
 (tv/create! {:name "Titanium" :language "Clojure"}))
```

A more detailed example: 

``` clojure
(ns myservice.graphs
  (:require [clojurewerkz.titanium.graph    :as tg]
            [clojurewerkz.titanium.vertices :as tv]))

(defn- main
  [& args]
  (tg/open (System/getProperty "java.io.tmpdir")) 
  (tg/transact!
      (tv/create! {:name "Michael" :location "Europe"})
      (tv/create! {:name "Zack"    :location "America"})))
```

### Creating Edges

Now that we know how to create vertices, we can begin creating edges
with the `clojurewerkz.titanium.edges/connect!` function: 

``` clojure
(require '[clojurewerkz.titanium.edges :as te])

(te/connect! v1 :connected v2))
```

Edges can have properties, just like vertices:

``` clojure
(require '[clojurewerkz.titanium.graph :as tg])
(require '[clojurewerkz.titanium.vertices :as tv])
(require '[clojurewerkz.titanium.edges :as te])

(tg/transact!
 (let [p1 (tv/create! {:title "ClojureWerkz" :url "http://clojurewerkz.org"})
       p2 (tv/create! {:title "Titanium"     :url "http://titanium.clojurewerkz.org"})]
   (te/connect! p1 :links p2 {:verified-on "February 11th, 2013"})))
```

## Basic graph theory 

Well, now we know how to create vertices and we know how to create
nodes. 

## Wrapping up

Congratulations, you now can use Titanium to perform fundamental graph
operations. Now you know enough to start building something real.
There are many features that we haven't covered here; they will be
explored in the rest of the guides.

We hope you find Titanium reliable, consistent and easy to use. In
case you need help, please ask on the
[mailing list](https://groups.google.com/forum/#!forum/clojure-titanium),
subscribe to [our blog](http://blog.clojurewerkz.org) and
[follow us on Twitter](http://twitter.com/ClojureWerkz).


## What to read next

The documentation is organized as a number of guides, covering all
kinds of topics.

We recommend that you read the following guides first, if possible, in
this order:

 * [Populating the graph](/articles/populating.html) [TBD]
 * [Querying and Traversing the graph](/articles/traversing.html) [TBD]


## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on
Twitter or the
[Titanium mailing list](https://groups.google.com/forum/#!forum/clojure-titanium)

Let us know what was unclear or what has not been covered. Maybe you do not like the guide style or grammar or discover spelling mistakes. Reader feedback is key to making the documentation better.
