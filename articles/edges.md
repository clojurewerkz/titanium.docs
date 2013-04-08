## Working with Edges 

[Good news everyone](http://www.youtube.com/watch?v=1D1cap6yETA)! If
you understood the section on working with vertices, then you already
understand about half of what it takes to work with edges.
`clojurewerkz.titanium.edges` contains analogs for `get`, `id-of`,
`to-map`, `assoc!`, `dissoc!`, `merge!`, `update!`, `clear!`, and
`remove!`. All of the previous functions do exactly the same thing as
for edges and vertices (with the exception that `to-map` returns a
 map with an extra `:__label__`  property).

