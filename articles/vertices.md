### Listing Property Names

To obtain a list of property names, use `clojurewerkz.titanium.vertices/keys`:

``` clojure
(require '[clojurewerkz.titanium.vertices :as tv])

(te/property-names v)
```


### Getting the id

To get the id of a vertex, use `clojurewerkz.titanium.vertices/id-of`:

``` clojure
(require '[clojurewerkz.titanium.vertices :as tv])

(tv/id-of v)
```

### Getting a Property

To get a single property value, use `clojurewerkz.titanium.vertices/get`:

``` clojure
(require '[clojurewerkz.titanium.vertices :as tv])

(tv/get v :name)
```

### Getting all Properties

Vertices in Titanium are *mutable objects*. However, it is easy to
obtain an immutable Clojure map of their properties using
`clojurewerkz.titanium.vertices/to-map`:

``` clojure
(require '[clojurewerkz.titanium.elements :as te])

(te/to-map v)
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
    (let [m {:name "Michael" :location "Europe"}
          v (tv/create! m)
          vm (tv/to-map v)]
          (= "Michael" (:name vm))
          (= "Europe"  (:location vm))
          ;; {:__id__ 40004, :name Michael, :location Europe}
          (println vm))))
```

Note that the returned maps will include all of the original
properties of the node, as well as an additional `:__id__` property
which holds onto the id of the node. 

### Setting Properties

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

### Merging Properties

To merge a map (or multiple maps) into the map of vertex properties,
use `clojurewerkz.titanium.vertices/merge!`:

``` clojure
(require '[clojurewerkz.titanium.vertices :as tv])

(tv/merge! v {:status "crawled" :crawled-at date})
```

### Updating Properties

To update a vertex's properties, use `clojurewerkz.titanium.vertices/update!`:

``` clojure
(require '[clojurewerkz.titanium.elements :as te])

(te/update! v :age inc)
```


### Unsetting Properties

To remove a property from a vertex, use `clojurewerkz.titanium.vertices/dissoc!`:

``` clojure
(require '[clojurewerkz.titanium.vertices :as tv])

(tv/dissoc! v :status :crawled-at)
```

To clear all properties on a vertex, there is `clojurewerkz.titanium.vertices/clear!`:

``` clojure
(require '[clojurewerkz.titanium.vertices :as tv])

(tv/clear! v)
```

### Removing Vertices

To remove a vertex, use `clojurewerkz.titanium.vertices/remove!`:

``` clojure
(require '[clojurewerkz.titanium.vertices :as tv])

(tv/remove! v)
```

### Retrieving Vertices

To remove a vertex, use `clojurewerkz.titanium.vertices/remove!`:

``` clojure
(require '[clojurewerkz.titanium.vertices :as tv])

(tv/find-by-kv! :name "Sarah Connor")
```
