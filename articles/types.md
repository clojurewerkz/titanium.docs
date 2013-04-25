---
title: "Types"
layout: article
---

## Working with types

Pretty soon after getting started with Titanium, graph theory will hit
you like a ton of bricks. All at once, you'll have a ton of ideas
about how to model the world. Walking down the street, you'll start
seeing graphs everywhere. Financial transactions, social networks,
traffic analysis, epidemiology, a few bridges in Germany and more will
boil down to a few lines on a blackboard once you let graph theory
permeate your thought processes.
[Types in Titan](https://github.com/thinkaurelius/titan/wiki/Type-Definition-Overview)
are meant to help you pursue these visions and help you better model
the world to the best of your ability.

There are two kinds of types: edge labels and property keys. There is
a small overlap in terms of functionality enabled between the two, but
for the most part each kind of type addresses a separate concern.
We'll start first with property keys as they will probably be the most
familiar to someone coming from a traditional database background.
Note that Titan automatically creates definitions for property keys
and edge labels if they are used before they are defined. This section
is about how to override the defaults and do more interesting things. 

Unless otherwise mentioned, all of the following functions are in
`clojurewerkz.titanium.types`. The examples are all being run in the
following namespace.

``` clojure
(ns titanium.intro
  (:require [clojurewerkz.titanium.graph    :as tg]
            [clojurewerkz.titanium.types    :as tt]))
```

### Retrieving a type

`get-type` takes in a keyword and, if such a type exists, returns that
type.

### Defining property keys

`defkey` takes in a keyword, class, and, optionally, a
map of attributes for the type. Here's a quick example:

``` clojure
(tg/transact! (tt/defkey :age Integer))
```

This defines a property key called `:age` that will be an Integer.
Let's see a slightly more complicated example:

```clojure
(tg/transact!       
 (tt/defkey :name String
            {:indexed-vertex? true
             :unique-direction :out}))
```

This defines a property key called `:name` which expects a string. In
addition, it expects a vertex's name to be unique and tells Titan to
create an index for the property. The terminology for
[the various kinds of uniqueness is still evolving](https://github.com/thinkaurelius/titan/issues/224),
but we've chosen to use Titan's wording for the time being.

Below is a list of the possible options that can be passed in to
configure the creation of the property key:

* `:indexed-edge?`, Boolean - whether this property should be indexed
  for edges.
* `:indexed-vertex?`, Boolean - whether this property should be
  indexed for vertices.
* `:unique-direction`, `:out` \ `:in` \ `:both` - `:out` means that a
  vertex can only have at most one of this property. `:in` unique
  means that only one vertex can have a given value for the property.
  `:both` means that the rules for `:out` and `:in` both apply.
* `:unique-locked`, Boolean - if the property is unique, this option
  determines whether a lock should be acquired during transactions
  (probably smart to leave this alone if you don't know what you are
  doing).


### Defining edge labels

`deflabel` takes in a keyword and, optionally, a
map of attributes for the label. Here's a quick example:

``` clojure
(tg/transact!
 (tt/deflabel :friends))
```

This simply creates a label called `:friends`. Let's see something a
bit more complicated:

``` clojure
(tg/transact!
 (tt/deflabel :heard-of {:direction "unidirected"                                
                         :unique-direction :out}))
```

The above creates a label called `:heard-of`. This label is
unidirected and out-unique, which means that each vertex has at most
one edge going out of it with this label. 

Below is a list of the possible options that can be passed in to
configure the creation of the edge label:

* `:direction`, `"unidirected"`\`"directed"` - specify the
  directionality of the edge.
* `:unique-direction`, `:out` - `:out` unique means that each vertex
has at most one edge going out of it with this label.
* `:unique-locked`, Boolean - if the property is unique, this option
  determines whether a lock should be acquired during transactions
  (probably smart to leave this alone if you don't know what you are
  doing).

### Serialization of types

Titan uses Kryo to store serialize types for storage. Look into
[how Titan does serialization](https://github.com/thinkaurelius/titan/wiki/Datatype-and-Attribute-Serializer-Configuration)
if you want to get really fancy.

### Types are forever

As of the writing of this documentation, once a type has been defined,
it can never be deleted or changed. Modifiable and removable types are
on the road map for Titan, but they just aren't there yet. That means
if you try to redefine a type when you restart a process, Titan will
throw errors. Titanium provides the idempotent methods
`defkey-once` and `deflabel-once`. These methods
will check to see if the given type exists and then, if it does not,
it will create the type. Otherwise it will do nothing. So, please
remember, types are forever (or until you clear your database). 
