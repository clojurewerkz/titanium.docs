---
title: "Configuring Titanium"
layout: article
---

## Configuring Titan

`clojurewerkz.titanium.graph/open` is a polymorphic function that
accepts:

 * Clojure maps where keys are configuration properties
 * Paths to configuration (`.properties`) files as strings or `java.io.File` instances
 * Paths to local DB directories as strings or `java.io.File` instances

It is a minimal wrapper on top of [the configuration methods for Titan
that already exist](https://github.com/thinkaurelius/titan/wiki/Graph-Configuration). 

### In-Memory Graph DB

In-memory graph can be obtained using the
`clojurewerkz.titanium.graph/open` function:

``` clojure
(ns myservice.graphs
  (:require [clojurewerkz.titanium.graph :as tg]))

(defn- main
  [& args]
  ;; opens an in-RAM graph database
  (tg/open {"storage.backend" "inmemory"})
 (comment Work with the graph here))
```


### BerkeleyDB

Here's one way of opening a Titan instance backed by BerkeleyDB. It's
useful for testing and REPL experimentation:

``` clojure
(ns myservice.graphs
  (:require [clojurewerkz.titanium.graph :as tg]))

(defn- main
  [& args]
  ;; opens a BerkeleyDB-backed graph database in a temporary directory
  (tg/open (System/getProperty "java.io.tmpdir"))
  (comment Work with the graph here))
```

Note that BerkeleyDB will lock directory it uses, so only one JVM can access a directory
at a time. If you have a BerkeleyDB-backed database open in the REPL, this may prevent
you from running tests or apps on the command line.

### Everything else

Most everything else can be accomplished by using a `.properties` file
or passing in a clojure map. Titanium steps out of your way and
doesn't provide anything extra on top of Titan. Here's what
[Bumi](https://github.com/zmaril/bumi) uses to configure Titan. This
opens up a Titan instance with embedded Cassandra by specifying the
backend and the config file for cassandra.

``` clojure
;;...

(def graph-config
  { ;; Embedded cassandra settings
   "storage.backend"  "embeddedcassandra"
   "storage.cassandra-config-dir" 
   (str "file://" (System/getProperty "user.dir")  "/resources/cassandra.yaml") }  )

(tg/open graph-config)
;;...
```


