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
