--- 
title: "Transaction Management" 
layout: article 
---

## Transaction Management

During the development of Archimedes and Titanium, a decision was made
to only support explict use of transactions. That means that errors
will be thrown if most functions from Titanium are used outside the
body of a `clojurewerkz.titanium.graph/transact!` call. In addition,
vertex and edge objects automatically go stale outside transactions
and must be refreshed to be reused in other transactions. Refreshing
is very simple and is accomplished via
`clojurewerkz.titanium.vertices/refresh!` and
`clojurewerkz.titanium.edges/refresh!`. This guide will dive into how
and why this style of transaction management is used.

### Transaction fundamentals 
If you've read the
[getting started guide](articles/getting_started.html), you'll already
have had exposure to `transact!`. Using `transact!` is
straightforward:

```clojure
(tg/transact! (tv/create!)) 
;; #<PersistStandardTitanVertexv[404]> 
``` 

Conceptually, think of `transact!` as a `do`. The result of
`transact!` will be the last s-expression evaluated inside the body.
If an error occurs, Titanium will let it be thrown and expects you to
deal with it.

### Complex transactions 

`transact!` is just a macro which binds a var inside of Titanium to be
a transaction and then takes care of the ceremony for executing the
transaction and persisting it to the database. That means we can do
all sorts of neat things inside transactions. The following
transaction uses [Ogre](/articles/ogre.html) and finds a node by name, looks for the
friends of the friends of the node, takes their names and provides does a
map containing the frequency of each name. 

``` clojure
(require '[ogre.core :as oq])

(tg/transact! 
    (oq/query (tv/find-by-kv :name "Zack")
              (oq/--> :friends)
              (oq/--> :friends)
              (oq/property :name)
              oq/into-vec!
              frequencies))
```

The main gotcha to watch out for is lazy evaluation. A simple `doall`
on results will take care of that. Otherwise, errors will be thrown
all over the place about how you are trying to access elements outside
of their transactional scope.

### Refreshing objects
    
After a vertex or edge is returned from a transaction, it becomes
stale. This means that before the object can be used again it must be
refreshed using either `clojurewerkz.titanium.vertices/refresh!` or
`clojurewerkz.titanium.edges/refresh!`.

``` clojure
(def stale-node (tg/transact! (tv/create!)))

(tg/transact! (tv/set-property! stale-node :name "Zack"))             ;; Shouldn't work!
(tg/transact! (tv/set-property! (tv/refresh stale-node) :name "Zack")) ;; Should work!
```

### Retrying transactions

While `transact!` and the two `refresh` methods might seem like
overkill compared to other transaction system, development and
reasoning about transactions becomes much easier. In particular, we
can retry transactions very simply with the
`clojurewerkz.titanium.graph/retry-transact!` method.
`retry-transact!` takes two arguments and the body of the
transactions. The first argument is a number indicating the number of
times to retry the transaction. The second argument is either a number
or a function which returns a number. This number will determine the
minimum amount of time Titanium will wait before retrying each transaction. 

``` clojure
(let [start-time (System/currentTimeMillis)]
        (tg/retry-transact! 3 100 
                            (println (float (/ (- (System/currentTimeMillis) start-time) 1000)))
                            (/ 1 0)))
```

Geometric backing off with small random offset:

``` clojure
(let [start-time (System/currentTimeMillis)
      back-off  (fn [try-count] (+ (Math/pow 10 try-count) 
                                (* try-count (rand-int 100))))]
        (tg/retry-transact! 4 back-off
                            (println (float (/ (- (System/currentTimeMillis) start-time) 1000)))
                            (/ 1 0)))
```
