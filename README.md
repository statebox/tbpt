WIP: https://github.com/ipfs/notes/issues/168

----

This note describes an encoding of (finite) [bipartite graphs](https://en.wikipedia.org/wiki/Bipartite_graph).

http://mathworld.wolfram.com/BipartiteGraph.html

# Motivation

Bi-partite graphs are ubiquitous in mathematics and computer science.

Matching on a graph is a very natural application (customer ↔ taxi), recommendations, clustering.
Lots of cool things.

And, of course, my personal motivation, http://statebox.github.io but more on that later ;^)

/cc @repatica @epost @whyrusleeping @jbenet @diasdavid 

## Details

A bi-partite graph is a graph where the nodes can be partitioned in two
disjoint sets, say `A` and `B`.

The edges only run *between* the two sets, so `A=>B` and `B=>A` only, no `B=>B` or `A=>A`. Hence, (directed) bi-partited graphs are often depicted as such:

<img width="185" alt="screen shot 2016-09-24 at 12 30 31" src="https://cloud.githubusercontent.com/assets/315734/18806095/b7e50d5c-8252-11e6-9abf-e53cd37612f4.png">

For clarity and nothing else, I used nrs for one partition and letters for the other.

## Proposed encoding

We now describe the graph by listing the adjacent edges of each node of one partition. So lets focus on the diamons and list them out.

<img width="217" alt="screen shot 2016-09-24 at 12 30 43" src="https://cloud.githubusercontent.com/assets/315734/18806098/c0eb120c-8252-11e6-8d02-b3efc43ee18d.png">

Now in the directed case, each node a *pair* of sets of edges, namely the in- and outgoing edges. When we list this out we get (JSON example):

```js
{ a: [ [1]  , [2] ]
, b: [ [1,2], []  ]
, c: [ []   , [2] ]
}
```

Now I do not want to name the nodes `a, b, c`, just be able to refer to them...
So instead we pick some order and infer their identifiers from the position in the array:

```json
[ [ [1]  , [2] ]
, [ [1,2], []  ]
, [ []   , [2] ]
]
```

## Result

Whats now left is:

> List of lists (pairs) of lists of numbers

Note that there is an upperbound to these numbers, namely the size of
the other set. So, you can minize the upper bound by picking the
partition with the most nodes.

Not only that but we can further simplify.

- Let the number 0 denote `[`
- Let the number 1 denote `]`
- Counting the nodes starting at 2 (so 0=>2, 1=>3, ...).

Our example now becomes (indented for clarity):

```
0
 0 0 2 1
   0 3 1 1
 0 0 2 3 1
   0 1 1
 0 0 1
   0 3 1 1
1
```

Thus we are left with

> List of numbers smaller than some upper bound.

This naturally leads to encoding as a list of `varints`, see [protcol buffers](https://developers.google.com/protocol-buffers/docs/encoding#varints) for more information. Compression algorithms should work really well on this.

We now have a compact binary encoding of bipartite graphs and a natural decoding into JSON.

## Further notes:

- Why not `n`-partite? Well 1) the edge set becomes more involved when n goes up and 2) they are used less often.

- What about undirected? Well, in that case you can leave out the *pair* altogether and stick to `[[n]]`. The parser can detect this by looking at how deep the lists are nested.

- How to add data to this? Well, you can just we use a labeling
  function, along the lines of

```js
{ labels: { vertices: { A: { 0: { someProp: 'someValue' },
                             1: { someProp: 'someOtherValue' } }
                      , B: { 0: { someOtherProp: 3.1415927 } } }
          , edges: { AB: {} , BA: {} }
          }
}
```

Opinions?!
