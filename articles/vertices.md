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

### Simple creation 

`create!` will create a vertex and, optionally, assign a map of
properties to it as well.

``` clojure
(tg/transact! (tv/create!))
(tg/transact! (tv/create! {:name "Bear"}))
```

### Upserting 

`upsert!` will look to see if vertices exist with the given key and
property. If so, then the function will merge the given map into the
found vertices and return the set of vertices. If not, the vertex will
be created with the supplied properties then put into a set and
returned.

``` clojure
(tg/transact! (tv/upsert! :location {:name "Zack" :location "Moscow"}))
(tg/transact! (tv/create! {:name "Trent" :location "Moscow"}))
(tg/transact! (tv/upsert! :location {:location "Moscow" :occupation "student"}))
```

### Uniquely upserting vertex 

`unique-upsert!` is exactly like `upsert!`, except it only returns one
vertex. It will throw an error if more than one vertex is returned. 
    
## Getting properties 

Below are methods which somehow deal with getting the properties of a
node.

### Getting a property

To get a single property value, use `get`:

``` clojure
(tg/transact! (tv/get example-vertex :name))
;; Zack
```

### Listing property names

To obtain a set of the property names for a node, use
`keys`:

``` clojure
(tg/transact! (tv/keys example-vertex))
;; #{:name :role}
```

### Listing property values

To obtain a set of the property values for a node, use
`vals`:

``` clojure
(tg/transact! (tv/vals example-vertex))
;; #{"Zack" "example"}
```

### Getting the id

To get the id of a vertex, use `id-of`:

``` clojure
(tv/id-of example-vertex)
;; 1337
```

### Getting all properties

Vertices in Titanium are *mutable objects*. However, it is easy to
obtain an immutable Clojure map of their properties using
`to-map`:

``` clojure
(tv/to-map example-vertex)
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
(tv/assoc! v :status "crawled" :crawled-at date)
```

### Merging properties

To merge a map (or multiple maps) into the map of vertex properties,
use `merge!`:

``` clojure
(tv/merge! v {:status "crawled" :crawled-at date})
```

### Updating properties

To update a vertex's properties, use `update!`:

``` clojure
(tv/update! v :age inc)
```


### Unsetting properties

To remove a property from a vertex, use `dissoc!`:

``` clojure
(tg/transact! (tv/dissoc! example-vertex :status :crawled-at))
```

### Clearing all properties
To clear all properties on a vertex, there is `clear!`:

``` clojure
(tg/transact! (tv/clear! example-vertex))
```

## Deleting vertices

To delete a vertex, use `delete!`:

``` clojure
(tg/transact! (tv/delete! example-vertex))
```

## Retrieving vertices

The methods below provide various ways to retrieve vertices. 

### Retrieving by id

Given an id, `find-by-id` retrieves the vertex with that idea. 

### Retrieving by key and value

Given a key and a value, `find-by-kv` returns all the vertices which
have that key and value in a set. 

### Retrieving all vertices

`get-all-vertices` returns all vertices. 
