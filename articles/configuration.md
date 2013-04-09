---
title: "Configuring Titanium"
layout: article
---

Titan configuration is typically file-based. `clojurewerkz.titanium.graph/open` is a polymorphic
function that accepts

 * Clojure maps where keys are configuration properties
 * Paths to configuration (`.properties`) files as strings or `java.io.File` instances
 * Paths to local DB directories as strings or `java.io.File` instances

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

To use a BerkeleyDB backend, pass a local database path as a string or
a path to a `.properties` file that
[configures Titan to use BerkeleyDB](https://github.com/thinkaurelius/titan/wiki/Using-BerkeleyDB)
to `clojurewerkz.titanium.graph/open`.

An example suitable for testing and REPL experimentation:

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

# Cassandra stuff
Keys that are relevant for the
Cassandra backend are

 * `"storage.backend"`: must be `"Cassandra"`)
 * `"storage.hostname"`: a host where a Cassandra node is running
 * `"storage.keyspace"`: keyspace Titan will use (default: `"titan"`)
 * `"storage.read-consistency-level"`: consistency level for reads (default: quorum)
 * `"storage.write-consistency-level"`: consistency level for writes (default: quorum)
 * `"storage.replication-factor"`: how many nodes will store replicas of the data. Default is 1, 3 recommended for
   production use.