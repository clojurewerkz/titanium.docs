---
title: "Working with Vertices"
layout: article
---

## Working with Vertices 

This guide provides a list of all methods for working with vertices in
Titanium. Unless otherwise mentioned, all of the following functions
reside in the `clojurewerkz.titanium.vertices` namespace.

All examples should run with the following namespace declaration.

``` clojure
(ns titanium.vertices
  (:require [clojurewerkz.titanium.graph    :as tg]
            [clojurewerkz.titanium.vertices :as tv]))
            
(tg/open (System/getProperty "java.io.tmpdir"))            

(def example-vertex (tg/transact! (tv/create! {:name "Zack" :role "Example"})))
```

## Creating vertices

`create!` will create a vertex and, optionally, assign a map of
properties to it as well.

```
(tg/transact! (tv/create!))
(tg/transact! (tv/create! {:name "Bear"}))
```

## Getting properties 

Below are methods which somehow deal with getting the properties of a
node.

### Getting a property

To get a single property value, use `clojurewerkz.titanium.vertices/get`:

``` clojure
(tg/transact! (tv/get v :name))
;; Zack
```

### Listing property names

To obtain a set of the property names for a node, use
`clojurewerkz.titanium.vertices/keys`:

``` clojure
(tg/transact! (te/keys v))
;; #{:name :role}
```

### Listing property values

To obtain a set of the property values for a node, use
`clojurewerkz.titanium.vertices/vals`:

``` clojure
(tg/transact! (te/vals v))
;; #{"Zack" "example"}
```

### Getting the id

To get the id of a vertex, use `clojurewerkz.titanium.vertices/id-of`:

``` clojure
(tv/id-of example-vertex)
;; 1337
```

### Getting all properties

Vertices in Titanium are *mutable objects*. However, it is easy to
obtain an immutable Clojure map of their properties using
`clojurewerkz.titanium.vertices/to-map`:

``` clojure
(te/to-map example-vertex)
;; {:name "Zack" :role "example" :__id__ 1337}
```

Note that the returned maps will include all of the original
properties of the node, as well as an additional `:__id__` property
which holds onto the id of the node. 

## Modifying properties 

### Setting properties

Because vertices in Titanium are mutable, functions that mutate their
properties are named with an exclamation point ("bang") at the end,
like
[Clojure transients](http://clojure-doc.org/articles/language/collections_and_sequences.html#transients).

To set one or more properties on a vertex, use
`clojurewerkz.titanium.vertices/assoc!`, which mimics
[clojure.core/assoc!](http://clojuredocs.org/clojure_core/clojure.core/assoc!):

``` clojure
(require '[clojurewerkz.titanium.vertices :as tv])

(tv/assoc! v :status "crawled" :crawled-at date)
```

### Merging properties

To merge a map (or multiple maps) into the map of vertex properties,
use `clojurewerkz.titanium.vertices/merge!`:

``` clojure
(require '[clojurewerkz.titanium.vertices :as tv])

(tv/merge! v {:status "crawled" :crawled-at date})
```

### Updating properties

To update a vertex's properties, use `clojurewerkz.titanium.vertices/update!`:

``` clojure
(require '[clojurewerkz.titanium.elements :as te])

(te/update! v :age inc)
```


### Unsetting properties

To remove a property from a vertex, use `dissoc!`:

``` clojure
(tg/transact! (tv/dissoc! example-vertex :status :crawled-at))
```

### Clearing all properties
To clear all properties on a vertex, there is `clojurewerkz.titanium.vertices/clear!`:

``` clojure
(tg/transact! (tv/clear! example-vertex))
```

## Deleting vertices

To delete a vertex, use `clojurewerkz.titanium.vertices/delete!`:

``` clojure
(tg/transact! (tv/delete! example-vertex))
```

### Retrieving vertices

