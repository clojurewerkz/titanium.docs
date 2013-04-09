---
title: "Getting Started with Titanium, a Clojure graph library"
layout: article
---

### Every journey begins with a single parentheses

This guide combines an overview of Titanium with a quick tutorial that
helps you to get started with it. It should take about 10 minutes to
read and study the provided code examples. This guide covers Titanium
1.0.x, specifically:

 * Features of Titanium
 * Clojure and Titan version requirements
 * How to add Titanium dependency to your project
 * A very brief introduction to graph databases and theory
 * How to create nodes and edges, fetch objects, and execute simple
   queries 
 * Applications of Titanium to simple graph theory problems

This work is licensed under a <a rel="license"href="http://creativecommons.org/licenses/by/3.0/">Creative Commons
Attribution 3.0 Unported License</a> (including images & stylesheets).
The source is available
[on Github](https://github.com/clojurewerkz/titanium.docs).


## What Titanium is

Titanium is a Clojure graph library built on top of
[Aurelius Titan](http://thinkaurelius.github.com/titan/). Titanium
strives to be easy to use, support all Titan features including the
various storage backends (Cassandra, HBase, BerkeleyDB Java Edition),
and takes the "batteries included" approach towards development.

To learn more about the Titan database, please see the following:

 * [The Rise of Big Graph Data](http://www.slideshare.net/slidarko/titan-the-rise-of-big-graph-data)
 * [Big Graph Data With Cassandra](http://www.youtube.com/watch?v=ZkAYA4Kd8JE)


## What Titanium is not

Titanium is not a database. It relies on other data store engines to
take care of on-disk durability. Titanium's value is in providing an
expressive Clojure API for graph operations.

Titanium is not an ORM/ODM. It does not provide graph visualization
features. Depending on the definition, Titanium may or may not be Web
Scale. Currently, the developers of Titanium are focused on
correctness and productivity and not benchmarks. 

## Supported Clojure versions

Titanium is built from the ground up for Clojure 1.4 and later. The most recent
stable release is always recommended.

## Adding Titanium Dependency To Your Project

### With Leiningen

    [clojurewerkz/titanium "1.0.0-alpha4"]

### With Maven

Add Clojars repository definition to your `pom.xml`:

```xml
<repository>
  <id>clojars.org</id>
  <url>http://clojars.org/repo</url>
</repository>
```


And then add the dependency:

``` xml
<dependency>
   <groupId>clojurewerkz</groupId>
    <artifactId>titanium</artifactId>
   <version>1.0.0-alpha4</version>
</dependency>
```

It is recommended to stay up-to-date with new versions. New releases
and important changes are announced
[@ClojureWerkz](http://twitter.com/ClojureWerkz).


## Brief Introduction to Graph Databases

A graph is a data structure that represents objects and the
connections between them. The objects being connected are called
"vertices" or "nodes" and connections are called "edges" or
"relationships". Vertices may have a multitude of different
properties. A vertex could be used to represent a person and so might
have properties for age, name, and gender. An edge links two vertices
and it too can have many different properties. If two vertices
represented two people, an edge connecting them could represent their
friendship and have properties that detailed the last time they spoke
and when the friendship started. In addition, there can be multiple
edges between any two given nodes and edges can have a direction
associated with them.

## Opening a Graph

Titanium can
[be configured](https://github.com/thinkaurelius/titan/wiki/Graph-Configuration)
to use
[many different durable storage backends](https://github.com/thinkaurelius/titan/wiki/Storage-Backend-Overview):

 * [BerkeleyDB](https://github.com/thinkaurelius/titan/wiki/Using-BerkeleyDB)
 * [Apache Cassandra](https://github.com/thinkaurelius/titan/wiki/Using-Cassandra)
 * [Apache HBase](https://github.com/thinkaurelius/titan/wiki/Using-HBase)

Titanium also allows for graphs to be stored in memory. Please note,
putting information you care about only in the in-memory storage is
usually a bad idea.

In this guide, we will be introducing some features of Titanium using
an embedded instance of BerkeleyDB. For information about how to use
other database backends, please see the [configuration guide](). To
start working with Titan, we will pass a map of configuration
properties to `clojurewerkz.titanium.graph/open`. The function will
use the Titan API to open up a new graph with the specified
configuration and store the resulting object in a dynamic var.


``` clojure
(ns titanium.intro
  (:require [clojurewerkz.titanium.graph :as tg]))

(defn- main
  [& args]
  ;; opens a BerkeleyDB-backed graph database in a temporary directory
  (tg/open (System/getProperty "java.io.tmpdir"))
  (comment Work with the graph here))
```

## Creating Vertices

Vertices are created using the
`clojurewerkz.titanium.vertices/create!` function. Please note that
`clojurewerkz.titanium.graph/transact!` must always be the context in
which any code touching the database is run. For more information,
please see the [transaction management guide]().


``` clojure
(require '[clojurewerkz.titanium.graph :as tv])

(tg/transact! 
 (tv/create! {:name "Titanium" :language "Clojure"}))
```

A more detailed example: 

``` clojure
(ns titanium.intro
  (:require [clojurewerkz.titanium.graph    :as tg]
            [clojurewerkz.titanium.vertices :as tv]))

(defn- main
  [& args]
  (tg/open (System/getProperty "java.io.tmpdir")) 
  (tg/transact!
      (tv/create! {:name "Michael" :location "Europe"})
      (tv/create! {:name "Zack"    :location "America"})))
```

## Creating Edges

Now that we know how to create vertices, we can begin creating edges
with the `clojurewerkz.titanium.edges/connect!` function:

``` clojure
(require '[clojurewerkz.titanium.edges :as te])

(tg/transact!
 (te/connect! v1 :connected v2))
```

Edges can have properties, just like vertices:

``` clojure
(tg/transact!
 (let [p1 (tv/create! {:title "ClojureWerkz" :url "http://clojurewerkz.org"})
       p2 (tv/create! {:title "Titanium"     :url "http://titanium.clojurewerkz.org"})]
   (te/connect! p1 :links p2 {:verified-on "February 11th, 2013"})))
```

## Indexing and retrieving vertices 

Titanium isn't just a write only database. The library provides a
variety of functions for indexing and finding various objects.
`clojurewerkz.titanium.vertices/find-by-kv` takes in a keyword and a
value and finds all of the vertices with the corresponding key/value
pair. `clojurewerkz.titanium.types/create-property-key` takes in a
keyword, a class, and, optionally, a map specifying how to configure
the type and then creates a
[corresponding type in the Titan database](https://github.com/thinkaurelius/titan/wiki/Type-Definition-Overview). 

```clojure
(require '[clojurewerkz.titanium.types :as tt])

(tg/transact! 
 (tt/create-property-key :age Integer {:indexed-vertex? true :unique-direction :out}))

(tg/transact! 
 (tv/create! {:name "Zack"   :age 22}))
(tg/transact! 
 (tv/create! {:name "Trent"  :age 22}))
(tg/transact! 
 (tv/create! {:name "Vivian" :age 19}))


(tg/transact!
 (tv/find-by-kv :age 22))
;;#{#<CacheVertex v[12]> #<CacheVertex v[16]>}
```

## Simple queries 

The final building block we need is to be able to ask questions about
the graph we are constructing. Simple queries are handled by functions
in the `clojurewerkz.titanium.query` namespace, while more complex
queries can be performed using [Ogre](). 

One of the most basic questions when working with graphs is
determining the number of edges a vertex has. This quantity is
commonly called the degree of the vertex. We can use various functions
from `clojurewerkz.titanium.query` to create a simple `degree-of`
function that provides the degree of a given vertex. 

``` clojure
(ns titanium.intro
  (:require [clojurewerkz.titanium.graph    :as tg]
            [clojurewerkz.titanium.edges    :as te]
            [clojurewerkz.titanium.vertices :as tv]
            [clojurewerkz.titanium.query    :as tq]))
            
(defn degree-of [v]
 (tq/count-edges v))

(defn- main
  [& args]
  (tg/open (System/getProperty "java.io.tmpdir")) 
  (tg/transact!
      (let [v1 (tv/create!)
            v2 (tv/create!)
            v3 (tv/create!)
            v4 (tv/create!)]
            (te/connect! v1 :link v2)
            (te/connect! v1 :link v3)
            (println (map degree-of [v1 v2 v3 v4])))))
            ;; (2 1 1 0)
```

## Basic graph theory 

Now we know how to create vertices and edges, find the previously
created nodes, and query to find the degree of the vertices. Which,
when you think about, covers, like, half of graph theory. Jokes aside,
we can already start doing some interesting experiments. We will
switch over to an in memory graph, since we don't much care about
saving any of these nodes. 

Let's calculate the average degree for the
[complete graph](http://en.wikipedia.org/wiki/Complete_graph):

``` clojure
(ns titanium.intro
  (:require [clojurewerkz.titanium.graph    :as tg]
            [clojurewerkz.titanium.edges    :as te]
            [clojurewerkz.titanium.vertices :as tv]
            [clojurewerkz.titanium.types    :as tt]
            [clojurewerkz.titanium.query    :as tq]))

(defn degree-of [v]
 (tq/count-edges v))

(defn average-degree []
 (tg/transact! 
     (let [vs (tv/find-by-kv :type "vertex")]
         (float (/ (->> vs (map degree-of) (reduce +)) (* 2 (count vs)))))))

(defn complete-graph [n]
  (tg/open {"storage.backend" "inmemory"})   
  (tg/transact!
      (tt/create-property-key :type String {:indexed-vertex? true :unique-direction :out})
      (let [vs (map #(tv/create! {:type "vertex" :i %}) (range n))]      
          (doseq [v vs w vs :when (not= v w)]
              (te/connect! v :link w)))))
         
(defn- main
  [& args]
  (doseq [i (range 1 10)]  
      (complete-graph i)
      (println i (average-degree))))
```

What's the expected maximum degree of a random graph generated with an
even chance of there being an edge between two nodes?

TODO: rebuild this to reuse the graph. Really slow, trying to create a
new graph each time. 
``` clojure
(defn max-degree []
 (tg/transact! 
     (apply max (map degree-of (tv/find-by-kv :type "vertex")))))

(defn random-graph [n]
  (tg/open {"storage.backend" "inmemory"})   
  (tg/transact!
      (tt/create-property-key :type String {:indexed-vertex? true :unique-direction :out})
      (let [vs (map #(tv/create! {:type "vertex" :i %}) (range n))]      
          (doseq [v vs w vs :when (and (not= v w) (> 0.5 (rand)))]
              (te/connect! v :link w)))))         
(defn- main
  [& args]
  (doseq [i (range 1 10)]  
      (let [results (for [j (range 10)] (do (random-graph i) (max-degree)))]
          (println i (float (/ (reduce + results) (count results)))))))
```

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
