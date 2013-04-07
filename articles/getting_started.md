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


## Titanium Overview

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
take care of on-disk durability.Titanium's value is in providing an
expressive Clojure API for graph operations.

Titanium is not an ORM/ODM. It does not provide graph visualization
features, although this is an interesting area to explore in the
future versions. Titanium may or may not be Web Scale and puts
correctness and productivity above sky high benchmarks.


## Supported Clojure versions

Titanium is built from the ground up for Clojure 1.3 and later. The most recent
stable release is always recommended.



## Adding Titanium Dependency To Your Project

### With Leiningen

    [clojurewerkz/titanium "1.0.0-alpha2"]

### With Maven

Add Clojars repository definition to your `pom.xml`:

{% gist 65642c4b53d26539e5f6 %}

And then the dependency:

    <dependency>
      <groupId>clojurewerkz</groupId>
      <artifactId>titanium</artifactId>
      <version>1.0.0-alpha2</version>
    </dependency>

It is recommended to stay up-to-date with new versions. New releases and important changes are announced [@ClojureWerkz](http://twitter.com/ClojureWerkz).


## A Very Short Intro To Graph Databases

Graph is a data structure that represents connections (or lack of
them) between things. Connected things are called "vertices" or
"nodes" and connections are called "edges" or
"relationships". Vertices may have properties (like person name or
age), same for edges (for example, when two people first met
each other). There may be more than one edge between two
nodes. Edges are directed (have a start and an end; for
example, Web pages link to each other).


## Opening a Graph

Titanium is built on top of Titan which can be configured, for example, to use different durable storage
backends:

 * [BerkeleyDB Java Edition](http://www.oracle.com/technetwork/database/berkeleydb/overview/index-093405.html)
 * [Apache Cassandra](http://cassandra.apache.org)
 * [Apache HBase](http://hbase.apache.org)

as well as in-RAM storage (used primarily for automated testing and experimentation in the REPL).

Titan configuration is typically file-based. `clojurewerkz.titanium.graph/open` is a polymorphic
function that accepts

 * Clojure maps where keys are configuration properties
 * Paths to configuration (`.properties`) files as strings or `java.io.File` instances
 * Paths to local DB directories as strings or `java.io.File` instances


### In-Memory Graph DB

In-memory graph can be obtained using the `clojurewerkz.titanium.graph/open-in-memory-graph` function:

``` clojure
(ns myservice.graphs
  (:require [clojurewerkz.titanium.graph :as tg]))

(defn- main
  [& args]
  ;; opens an in-RAM graph database
  (let [g (tg/open-in-memory-graph)]
    (comment Work with the graph here)))
```


### Local Graph DB

To use BerkeleyDB backend, pass a local database path as a string or a path to
a `.properties` file that [configures Titan to use BerkeleyDB](https://github.com/thinkaurelius/titan/wiki/Using-BerkeleyDB)
to `clojurewerkz.titanium.graph/open`.

An example suitable for testing and REPL experimentation:

``` clojure
(ns myservice.graphs
  (:require [clojurewerkz.titanium.graph :as tg]))

(defn- main
  [& args]
  ;; opens a BerkeleyDB-backed graph database in a temporary directory
  (let [g (tg/open (System/getProperty "java.io.tmpdir"))]
    (comment Work with the graph here)))
```

Note that BerkeleyDB will lock directory it uses, so only one JVM can access a directory
at a time. If you have a BerkeleyDB-backed database open in the REPL, this may prevent
you from running tests or apps on the command line.

### Cassandra-backed Graph DB

To use BerkeleyDB backend, pass a path to
a `.properties` file that [configures Titan to use Cassandra](https://github.com/thinkaurelius/titan/wiki/Using-Cassandra)
or a Clojure map of configuration properties. Keys that are relevant for the Cassandra
backend are

 * `"storage.backend"`: must be `"Cassandra"`)
 * `"storage.hostname"`: a host where a Cassandra node is running
 * `"storage.keyspace"`: keyspace Titan will use (default: `"titan"`)
 * `"storage.read-consistency-level"`: consistency level for reads (default: quorum)
 * `"storage.write-consistency-level"`: consistency level for writes (default: quorum)
 * `"storage.replication-factor"`: how many nodes will store replicas of the data. Default is 1, 3 recommended for
   production use.

An example:

``` clojure
(ns myservice.graphs
  (:require [clojurewerkz.titanium.graph :as tg]))

(defn- main
  [& args]
  (let [g (tg/open {"storage.backend"  "Cassandra"
                    "storage.hostname" "127.0.0.1"
                    "storage.keyspace" "myapp_development"})]
    (comment Work with the graph here)))
```

For more information, consult [Titan documentation on the Cassandra backend](https://github.com/thinkaurelius/titan/wiki/Using-Cassandra).

### HBase-backed Graph DB

TBD


## Creating Vertices

Vertices are created using the `clojurewerkz.titanium.graph/add-vertex` function:

``` clojure
(require '[clojurewerkz.titanium.graph :as tg])

(tg/add-vertex g {:name "Titanium" :language "Clojure"})
```

The same example in context:

``` clojure
(ns myservice.graphs
  (:require [clojurewerkz.titanium.graph :as tg]))

(defn- main
  [& args]
  ;; in-RAM graph database used here only for demonstration
  (let [g (tg/open-in-memory-graph)
        v (tg/add-vertex g)]
    (comment ...)))
```

Vertices typically have properties. They are passed to `clojurewerkz.titanium.graph/add-vertex` as maps:

``` clojure
(require '[clojurewerkz.titanium.graph :as tg])

(tg/add-vertex g {:name "Titanium" :language "Clojure"})
```


## Working With Vertex Properties

### Getting a Map of Properties

Vertices in Titanium are *mutable objects*. However, it is easy to obtain an immutable Clojure map of
their properties using `clojurewerkz.titanium.elements/properties-of`:

``` clojure
(require '[clojurewerkz.titanium.elements :as te])

(te/properties-of v)
```

Example in context:

``` clojure
(ns myservice.graphs
  (:require [clojurewerkz.titanium.graph    :as tg]
            [clojurewerkz.titanium.elements :as te]))

(defn- main
  [& args]
  ;; in-RAM graph database used here only for demonstration
  (let [g (tg/open-in-memory-graph)
        v (tg/add-vertex g {:name "Titanium" :language "Clojure"})]
    ;= {"name" "Titanium" "language" "Clojure"}
    (te/properties-of v)))
```

Note that returned maps always have strings as keys.

### Getting A Single Property

To get a single property value, use `clojurewerkz.titanium.elements/property-of`:

``` clojure
(require '[clojurewerkz.titanium.elements :as te])

(te/property-of v :name)
```

To get id of a vertex, use `clojurewerkz.titanium.elements/id-of`:

``` clojure
(require '[clojurewerkz.titanium.elements :as te])

(te/id-of v)
```

### Listing Property Names

To obtain a list of property names, use `clojurewerkz.titanium.elements/property-names`:

``` clojure
(require '[clojurewerkz.titanium.elements :as te])

(te/property-names v)
```

### Mutating Properties

Because vertices in Titanium are mutable, functions that mutate their properties are named
with an exclamation point ("bang") at the end, like [Clojure transients](http://clojure-doc.org/articles/language/collections_and_sequences.html#transients).

To mutate one or more properties on a vertex, use `clojurewerkz.titanium.elements/assoc!`, which
mimics [clojure.core/assoc!](http://clojuredocs.org/clojure_core/clojure.core/assoc!):

``` clojure
(require '[clojurewerkz.titanium.elements :as te])

(te/assoc! v "status"     "crawled"
             "crawled-at" date)
```

To merge a map (or multiple maps) into the map of vertex properties, use `clojurewerkz.titanium.elements/merge!`:

``` clojure
(require '[clojurewerkz.titanium.elements :as te])

(te/merge! v {"status"    "crawled"
             "crawled-at" date})
```


### Unsetting Properties

To clear a property (set its value to `nil`), use `clojurewerkz.titanium.elements/dissoc!`:

``` clojure
(require '[clojurewerkz.titanium.elements :as te])

(te/dissoc! v "status" "crawled-at")
```

To clear all properties on a vertex, there is `clojurewerkz.titanium.elements/clear!`:

``` clojure
(require '[clojurewerkz.titanium.elements :as te])

(te/clear! v)
```

## Removing Vertices

To remove a vertex, use `clojurewerkz.titanium.graph/remove-vertex` which takes
a graph instance and a vertex:

``` clojure
(require '[clojurewerkz.titanium.graph :as tg])

(let [v (tg/add-vertex g {:name "Titanium" :language "Clojure"})]
  (tg/remove-vertex g v))
```


## Creating Edges

Now that we know how to create vertices, lets create two vertices representing two Web pages that link to each other and add a directed
edge between them. To do that, we need to use the `clojurewerkz.titanium.graph/add-edge` function:

``` clojure
(require '[clojurewerkz.titanium.graph :as tg])

(let [p1 (tg/add-vertex g {:title "ClojureWerkz" :url "http://clojurewerkz.org"})
      p2 (tg/add-vertex g {:title "Titanium"     :url "http://titanium.clojurewerkz.org"})]
  (tg/add-edge g p1 p2 "links"))
```

Edges can have properties, just like vertices:

``` clojure
(require '[clojurewerkz.titanium.graph :as tg])

(let [p1 (tg/add-vertex g {:title "ClojureWerkz" :url "http://clojurewerkz.org"})
      p2 (tg/add-vertex g {:title "Titanium"     :url "http://titanium.clojurewerkz.org"})]
  (tg/add-edge g p1 p2 "links" {:verified-on "February 11th, 2013"}))
```



## Working With Edge Properties

### Getting a Map of Properties

Just like vertices, edges in Titanium are *mutable objects*. To obtain an immutable Clojure map of
their properties, use the same function, `clojurewerkz.titanium.elements/properties-of`:

``` clojure
(require '[clojurewerkz.titanium.elements :as te])

(te/properties-of e)
```

Example in context:

``` clojure
(ns myservice.graphs
  (:require [clojurewerkz.titanium.graph    :as tg]
            [clojurewerkz.titanium.elements :as te]))

(defn- main
  [& args]
  ;; in-RAM graph database used here only for demonstration
  (let [g  (tg/open-in-memory-graph)
        p1 (tg/add-vertex g {:title "ClojureWerkz" :url "http://clojurewerkz.org"})
        p2 (tg/add-vertex g {:title "Titanium"     :url "http://titanium.clojurewerkz.org"})
        e  (tg/add-edge g p1 p2 "links" {:verified-on "February 11th, 2013"})]
    ;= {"verified-on" "February 11th, 2013"}
    (te/properties-of e)))
```

Note that returned maps always have strings as keys.

### Getting A Single Property

To get a single property value, use the same function as with verties,
`clojurewerkz.titanium.elements/property-of`:

``` clojure
(require '[clojurewerkz.titanium.elements :as te])

(te/property-of e :verified-on)
```

To get id of an edge, use `clojurewerkz.titanium.elements/id-of`:

``` clojure
(require '[clojurewerkz.titanium.elements :as te])

(te/id-of e)
```

### Listing Property Names

It is possible to list edge property names using `clojurewerkz.titanium.elements/property-names`:

``` clojure
(require '[clojurewerkz.titanium.elements :as te])

(te/property-names e)
```

### Mutating Properties

Edge properties are mutated exactly the same way as vertex properties.

To mutate one or more properties on a vertex, use `clojurewerkz.titanium.elements/assoc!`, which
mimics [clojure.core/assoc!](http://clojuredocs.org/clojure_core/clojure.core/assoc!):

``` clojure
(require '[clojurewerkz.titanium.elements :as te])

(te/assoc! e "verified-on" date)
```

To merge a map (or multiple maps) into the map of edge properties, use `clojurewerkz.titanium.elements/merge!`:

``` clojure
(require '[clojurewerkz.titanium.elements :as te])

(te/merge! e {"verified-on" date})
```


### Unsetting Properties

To clear a property (set its value to `nil`), use `clojurewerkz.titanium.elements/dissoc!`:

``` clojure
(require '[clojurewerkz.titanium.elements :as te])

(te/dissoc! e "status" "verified-on")
```

To clear all properties on an edge, there is `clojurewerkz.titanium.elements/clear!`:

``` clojure
(require '[clojurewerkz.titanium.elements :as te])

(te/clear! e)
```


## Removing Edges

To remove an edge, use `clojurewerkz.titanium.graph/remove-edge` which takes
a graph instance and an edge:

``` clojure
(require '[clojurewerkz.titanium.graph :as tg])

(let [p1 (tg/add-vertex g {:title "ClojureWerkz" :url "http://clojurewerkz.org"})
      p2 (tg/add-vertex g {:title "Titanium"     :url "http://titanium.clojurewerkz.org"})
      e  (tg/add-edge g p1 p2 "links")]
  (tg/remove-edge g e))
```


## Fetching Vertices

To find a vertex by id (if it is known), use `clojurewerkz.titanium.graph/get-vertex`:

``` clojure
(require '[clojurewerkz.titanium.graph    :as tg])
(require '[clojurewerkz.titanium.elements :as te])

(let [g  (tg/open-in-memory-graph)
      v1 (tg/add-vertex g {:age 28 :name "Michael"})
      v2 (tg/add-vertex g {:age 26 :name "Alex"})]
    (println (tg/get-vertex g (te/id-of v1))))
```

To find vertices by a key/value pair of properties, there is `clojurewerkz.titanium.graph/get-vertices`:

``` clojure
(require '[clojurewerkz.titanium.graph :as tg])

(let [g  (tg/open-in-memory-graph)
      v1 (tg/add-vertex g {:age 28 :name "Michael"})
      v2 (tg/add-vertex g {:age 26 :name "Alex"})
      xs (set (tg/get-vertices g "name" "Alex"))]
    (println xs))
```

To find all the graph vertices, use `clojurewerkz.titanium.graph/get-vertices` with just
one argument, the graph:

``` clojure
(require '[clojurewerkz.titanium.graph :as tg])

(let [g  (tg/open-in-memory-graph)
      v1 (tg/add-vertex g {:age 28 :name "Michael"})
      v2 (tg/add-vertex g {:age 26 :name "Alex"})
      xs (set (tg/get-vertices g))]
    (println xs))
```


## Fetching Edges

The API part that finds edges is very similar to the one covered above that works
with vertices.

To find an edge by id (if it is known), use `clojurewerkz.titanium.graph/get-edge`:

``` clojure
(require '[clojurewerkz.titanium.graph    :as tg])
(require '[clojurewerkz.titanium.elements :as te])

(let [g  (tg/open-in-memory-graph)
      v1 (tg/add-vertex g {:age 28 :name "Michael"})
      v2 (tg/add-vertex g {:age 26 :name "Alex"})
      e  (tg/add-edge g v1 v2 "friend")]
    (println (tg/get-edge g (te/id-of e))))
```

To find edges by a key/value pair of properties, use `clojurewerkz.titanium.graph/get-vertices`:

``` clojure
(require '[clojurewerkz.titanium.graph :as tg])

(let [g  (tg/open-in-memory-graph)
      v1 (tg/add-vertex g {:age 28 :name "Michael"})
      v2 (tg/add-vertex g {:age 26 :name "Alex"})
      e  (tg/add-edge g v1 v2 "friend" {:since 2008})
      xs (set (tg/get-edges g "since" 2008))]
    (println xs))
```

To find all the graph edges, use `clojurewerkz.titanium.graph/get-vertices` with just
one argument:

``` clojure
(require '[clojurewerkz.titanium.graph :as tg])

(let [g  (tg/open-in-memory-graph)
      v1 (tg/add-vertex g {:age 28 :name "Michael"})
      v2 (tg/add-vertex g {:age 26 :name "Alex"})
      e  (tg/add-edge g v1 v2 "friend" {:since 2008})
      xs (set (tg/get-edges g))]
    (println xs))
```


## Persisting Changes: Transaction Control

Every graph operation in Titan occurs within the context of a transaction. Usually there
is [no need to explicitly start a transaction](https://github.com/thinkaurelius/titan/wiki/Transaction-Handling): Titan will automatically start one
tied to the current thread on the first operation.

Transactions that happen across multiple threads (when the graph is populated or
updated from multiple threads) need to be started explicitly.

### Committing Changes

To persist changes to a durable storage (Cassandra, BerkeleyDB, etc), a transaction needs to be committed.
In Titanium, this is done via the `clojurewerkz.titanium.graph/commit-tx!` function:

``` clojure
(require '[clojurewerkz.titanium.graph :as tg])

(let [g  (tg/open-in-memory-graph)
      v1 (tg/add-vertex g {:age 28 :name "Michael"})
      v2 (tg/add-vertex g {:age 26 :name "Alex"})
      e  (tg/add-edge g v1 v2 "friend" {:since 2008})]
    ;; commits changes to durable storage
    (tg/commit-tx! g)
```

### Automatic Commit On Shutdown

Closing a graph with `clojurewerkz.titanium.graph/close` will commit
the current automatically started transaction (if any).


### Rolling Back Changes

To roll back a transaction, use `clojurewerkz.titanium.graph/rollback-tx!`:

``` clojure
(require '[clojurewerkz.titanium.graph :as tg])

(let [g  (tg/open-in-memory-graph)
      v1 (tg/add-vertex g {:age 28 :name "Michael"})
      v2 (tg/add-vertex g {:age 26 :name "Alex"})
      e  (tg/add-edge g v1 v2 "friend" {:since 2008})]
    ;; rolls back changes
    (tg/rollback-tx! g)
```


### Explicit Transactions

Transactions can be started explicitly using `clojurewerkz.titanium.graph/start-x` which
takes a graph instance. The returned transaction then should be used instead of the graph
instance with all graph operations (e.g. `clojurewerkz.titanium.graph/add-vertex`):

``` clojure
(let [g   (tg/open "/tmp/titanium/graph.db")
      tx  (tg/start-tx g)]
    (tg/add-vertex tx {})
    (tg/commit-tx! tx)
    (tg/close g))
```

Alternatively, `clojurewerkz.titanium.graph/run-transactionally` takes a graph and a function
and yields a newly started transaction to it:

``` clojure
(let [g   (tg/open "/tmp/titanium/graph.db")]
    (tg/run-transactionally g (fn [tx]
      (tg/add-vertex tx {})))
    (tg/close g))
```

If an exception occurs during the execution of the function, the transaction will be
rolled back. Otherwise it is committed after the function returns.

Explicitly started transactions can be shared between threads. It is not necessary to use
explicit transactions when only one thread modifies a graph and nested transactions are
not necessary.


### Transactions and In-memory Graphs

Note that in-memory graphs do not support transactions.


## Closing a Graph

To close a graph, use `clojurewerkz.titanium.graph/close`. It will flush all pending changes
to durable storage. Closed graphs can no longer be used again.

Graphs can be closed when an app is shutting down or after processing a batch of
work.



## Wrapping up

Congratulations, you now can use Titanium to perform fundamental graph
operations. Now you know enough to start building a real
application. There are many features that we haven't covered here;
they will be explored in the rest of the guides.

We hope you find Titanium reliable, consistent and easy to use. In case you need help, please ask on the [mailing list](https://groups.google.com/forum/#!forum/clojure-titanium),
subscribe to [our blog](http://blog.clojurewerkz.org) and [follow us on Twitter](http://twitter.com/ClojureWerkz).


## What to read next

The documentation is organized as a number of guides, covering all kinds of topics.

We recommend that you read the following guides first, if possible, in this order:

 * [Populating the graph](/articles/populating.html) [TBD]
 * [Querying and Traversing the graph](/articles/traversing.html) [TBD]


## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on Twitter or the [Titanium mailing list](https://groups.google.com/forum/#!forum/clojure-titanium)

Let us know what was unclear or what has not been covered. Maybe you do not like the guide style or grammar or discover spelling mistakes. Reader feedback is key to making the documentation better.
