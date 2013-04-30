---
title: "Ogre integration"
layout: article
---

## Ogre integration 

Titanium has been built in parallel with
[Ogre](http://ogre.clojurewerkz.org/), a library designed to make
asking rich questions about graphs as easy as possible. Below we will
walk through one of the tests of Titanium and see what sorts of neat
things Ogre can do. This is by no means a comprehensive guide to Ogre,
but rather a means to entice you into using that library as well.

### Setup

First, we'll switch into an appropriate namespace and open up the database:

``` clojure
(ns clojurewerkz.titanium.integration-test
  (:require [clojurewerkz.titanium.graph    :as tg]
            [clojurewerkz.titanium.vertices :as tv]
            [clojurewerkz.titanium.edges    :as ted]
            [ogre.core :as oq])
  (:import java.io.File))

(tg/open (System/getProperty "java.io.tmpdir"))            
```

Note that Titanium is built with Ogre, and so `ogre.core` should be
immediately available once you have Titanium as a dependency. Next,
we'll create a simplified version of the [canonical graph of the gods
example](https://github.com/thinkaurelius/titan/wiki/Getting-Started)
:

``` clojure
(tg/transact!
 (let [saturn   (tv/create! {:name "Saturn"   :type "titan"})
       jupiter  (tv/create! {:name "Jupiter"  :type "god"})
       hercules (tv/create! {:name "Hercules" :type "demigod"})
       alcmene  (tv/create! {:name "Alcmene"  :type "human"})
       neptune  (tv/create! {:name "Neptune"  :type "god"})
       pluto    (tv/create! {:name "Pluto"    :type "god"})
       sea      (tv/create! {:name "Sea"      :type "location"})
       sky      (tv/create! {:name "Sky"      :type "location"})
       tartarus (tv/create! {:name "Tartarus" :type "location"})
       nemean   (tv/create! {:name "Nemean"   :type "monster"})
       hydra    (tv/create! {:name "Hydra"    :type "monster"})
       cerberus (tv/create! {:name "Cerberus" :type "monster"})]
   (ted/connect! neptune :lives sea)
   (ted/connect! jupiter :lives sky)
   (ted/connect! pluto :lives tartarus)
   (ted/connect! jupiter :father saturn)
   (ted/connect! hercules :father jupiter)
   (ted/connect! hercules :mother alcmene)
   (ted/connect! jupiter :brother pluto)
   (ted/connect! pluto :brother jupiter)
   (ted/connect! neptune :brother pluto)
   (ted/connect! pluto :brother neptune)
   (ted/connect! jupiter :brother neptune)
   (ted/connect! neptune :brother jupiter)
   (ted/connect! cerberus :lives tartarus)
   (ted/connect! pluto :pet cerberus)
   (ted/connect! hercules :battled nemean   {:times 1})
   (ted/connect! hercules :battled hydra    {:times 2})
   (ted/connect! hercules :battled cerberus {:times 12})))
```

### Who is Saturn's grandson? 

``` clojure
(tg/transact! 
 (oq/query (tv/find-by-kv :name "Saturn")
           (oq/<-- [:father])
           (oq/<-- [:father])
           (oq/property :name)
           oq/first-of!))
;= "Hercules" 
```          

### Who were the parents of Hercules? 

``` clojure
(tg/transact!
 (oq/query (tv/find-by-kv :name "Hercules")
           (oq/--> [:father :mother])
           (oq/property :name)
           oq/into-set!))
;= #{"Alcmene" "Jupiter"}           
```

### What has Hercules battled more than once?

``` clojure
(tg/transact! 
 (oq/query (tv/find-by-kv :name "Hercules")
           (oq/--E> [:battled])
           (oq/has :times > 1)
           (oq/in-vertex)
           (oq/property :name)
           oq/into-set!))
;= #{"Hydra" "Cerberus"}           
```

### How many different things has Hercules battled? 

``` clojure
(tg/transact!
 (oq/query (tv/find-by-kv :name "Hercules")
           (oq/--E> [:battled])
           oq/count!))
;= 3
```

### Who lives where Pluto lives?

``` clojure
(tg/transact!
 (let [pluto (first (tv/find-by-kv :name "Pluto"))]
   (oq/query pluto
    (oq/--> [:lives])
    (oq/<-- [:lives])
    (oq/except [pluto])
    (oq/property :name)
    oq/into-set!)))
;= #{"Cerberus"}    
```    

### Who are the brothers of Pluto and where do they live? 

``` clojure
(tg/transact! 
 (oq/query (tv/find-by-kv :name "Pluto")
           (oq/--> [:brother])
           (oq/as  "god")
           (oq/out [:lives])
           (oq/as  "place")
           (oq/select (oq/prop :name))
           oq/all-into-maps!
           set))
;= #{ {:god "Neptune" :place "Sea"} {:god "Jupiter" :place "Sky"}}
```

## Wrapping Up

That's just a taste of what Ogre can do. If you made it all the way
here, you should really
[read the docs](http://ogre.clojurewerkz.org/).


## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on
Twitter or the [Titanium mailing
list](https://groups.google.com/forum/#!forum/clojure-titanium)

Let us know what was unclear or what has not been covered. Maybe you
do not like the guide style or grammar or discover spelling
mistakes. Reader feedback is key to making the documentation better.
