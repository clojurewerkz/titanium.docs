---
title: "Working with edges"
layout: article
---

## Working with Edges 

[Good news everyone](http://www.youtube.com/watch?v=1D1cap6yETA)! If
you understood the section on
[working with vertices](/articles/vertices.md), then you already
understand about half of what it takes to work with edges.
`clojurewerkz.titanium.edges` contains analogs for `get`,
`keys`,`id-of`, `to-map`, `assoc!`, `dissoc!`, `merge!`, `update!`,
`clear!`,`refresh`,`delete!`, and `find-by-id`. All of the previous
functions do exactly the same thing for edges as they do for vertices
(with the exception that `to-map` returns a map with an extra
`:__label__` property). The functions below are unique to
`clojurewerkz.titanium.edges`. 

``` clojure
(ns titanium.intro
  (:require [clojurewerkz.titanium.graph    :as tg]
            [clojurewerkz.titanium.vertices :as tv]
            [clojurewerkz.titanium.edges    :as te]))
            
(tg/open (System/getProperty "java.io.tmpdir"))            

(def Zack (tg/transact! (tv/create! {:name "Zack"})))
(def Brooke (tg/transact! (tv/create! {:name "Brooke"})))
(def familial-tie (tg/transact! (te/connect! (tv/refresh Zack)
                                             :brother-to 
                                             (tv/refresh Brooke))))
```

## Getting edge label
`label-of` returns the label of an edge. 

``` clojure
(tg/transact! (te/label-of (te/refresh familial-tie)))
;;:brother-to
```
## Creation

Below are functions for creating edges between vertices. 

### Simple connect

`connect!` takes in a vertex, a label, another vertex and, optionally,
a map of properties. The function creates an edge that points from the
first vertex to the second vertex with the given label and properties. 

``` clojure
(tg/transact! (te/connect! (tv/refresh Zack)
                           :brother-to 
                           (tv/refresh Brooke)))
```

### Upconnect

`upconnect!` is a rough analog to
[`upsert!`](/articles/vertices.html#upserting). Given the same
arguments as `connect!`, `upconnect!` checks to see if there are any
edges with the given label between the given vertices. If so, it
updates the found edges with the new properties. If not, it creates
the edge and assigns the given properties to it.

### Unique upconnect

`unique-upconnect!` does the same thing as `upconnect!`, but only
returns one edge. If more than one edge is returned, an error is
thrown.

## Retrieve all edges

`get-all-edges` returns all the edges in the graph.