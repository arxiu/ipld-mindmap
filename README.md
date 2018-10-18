# IPLD Mindmap
_[Current live version](https://arxiu.github.io/ipld-mindmap-pts-render)_

## Context
We're in a personal contest to understand and try to change how do we organize our own information, and how it relates to others. The full explanation of why and how is something we're working on.

Since the scope of the project is inmense, we're going to tackle small projects to get insight, while creating new tools to allow new ways to relate to information.

The limitations around organizing information in a way that is meaningful for the individual has been one of the driving frustrations that has triggered us to digg deep into these issues. At the same time we believe that defining how information is organized and structured is a necessary buliding block before exploring more advance systems.

This is the first "project" we try to tackle. While creating a tool is the main goal, is likley that we will also be exploring working frameworks and different documentation praxis.

Documentation is important to us, because we are looking to solve problems in the best way possible, so if the logic behind is not good enough, we want to know. And probably you can help! We invite you to open an issue so we can discuss any question.

## Mindmap
We like the idea of mindmap, a tool that allows to organize information in the way that your brain works and not in a simplified way restricted by a user interface or a data structure.

[The wikpedia entry](https://en.wikipedia.org/wiki/Mind_map) says that mindmaps are hierachical and around a single concept. We're not making a reference to this exact concept, although it shares a lot of points in common.

## Original specs

-  Create a tool that would allow to represent what you could do in a analog mindmap, but in a digital format.
-  The main goal is to design and justifiy the correct data structure.
    - It needs to work on a global domain. This means that two different mindmaps pointing to the same concept should converge if put together
    -  It extends [IPLD](https://ipld.io/)
    -  As simple as possible
    -  As generic as possible (can cover as many usecases as possible)
    -  Any type of data should be able to be referenced
-  Eventually we'll explore authorship, accessibility, networking... but not yet.
-  It should have some basic visualization
-  The tool is render agnostic. Diferent renders can be eventually used.
-  We should document the process and the reasoning behind as close as possible
-  It should work on the web. Because of ease of use and development.
-  MVP approach. Keep things lean.
-  Nice to have
    -  Load content and render via IPFS
    -  Compatible with any IPFS/IPLD object

Deadline: end of September 2018

## Data structure

### Nodes in the global domain
For _Global domain_ we understand that there is only one single giant mindmap.

Which means that we need to be able to break in down into smaller subsets/pieces. The atomic piece of a mindmap is a node (which is a very conflicting name that we would like to change)

A node is nothing but a set of `relations` around a concept/content/idea. We call this "concept" `origin`.
So the `origin` is the data that represents where the `relations` are coming from.

These relations are towards another piece of data. From the `node` perspective, we call these data `targets`

**This implies that each `node` needs to contain and describe the `relations` with all the other `targets` it is interested, because if broken apart, it will loose information it cares about.**

In other words, `relations` come out of the `origin`, **never** in (at least from the `origin` perspective). If that was the case we will have a duplicated source of truth (the `relation` from the incoming `origin` and the `relation` from the current `origin`). This does not mean that the relation can't represent an incoming direction.

It makes a node behave selfishly, which is the logical behaviour in a distributed system.

Since we're in a global domain we will need to represent a subset of the global mindmap. Which will be just a lists of nodes.


### Relationships and nodes
We understand `relation` as how a piece of data (`origin`) relates to another peice of data (`target`). A node can have an arbitrary number of `relations`.

```json
    {
        "origin":"content1",
        "relations": [
            {"target": "content2"}
            {"target": "content3"}
        ]
    }

```

### Relationship types

One of our frustrations and things we are exploring in detail, is how can we extend how do we relate to information beyond what a user interface or the underlying system allows.

In this case, it translates on allowing the user to definie how a piece of information relates to another. So a `relation` can have a `type`, which is nothing but a reference to an expression of the `type` of relation.

A `type` of a `relation` could be "depends on", "is my dad", "contains"... or anything (text or not). It is the job of the `render` to understand what this `type` means and how to represent it.

Because the selfish behaviour of a node described above, a reltionship is always described from the perspective of the `origin` towards the `target`

```json
[
    {
        "origin":"Son",
        "relations": [
            {
                "target": "Dad",
                "type": "Is my dad"
            }
        ]
    },
    {
        "origin":"Dad",
        "relations": [
            {
                "target": "Son",
                "type": "Is my son"
            }
        ]
    }
]
```
The selfhish behaviour implies that there is no way to guarantee the integrity of the information, since the nodes could express conflicting information. And that's ok because **each node have it's own truth**.

```json
[
    {
        "origin":"Son",
        "relations": [
            {
                "target": "Dad",
                "type": "Is NOT my dad"
            }
        ]
    },
    {
        "origin":"Dad",
        "relations": [
            {
                "target": "Son",
                "type": "Is my son"
            }
        ]
    },
]
```

### Merkle paths as identifiers
In the previous examples we've used sample text in the `origin` field, to uniquely identifiy a piece of data.
This was just used for explanation purposes. It does not make sense in a global domain. And identifier needs to be global.

We first thought about using the [CID](https://github.com/ipld/cid) of the content. This is basically its hashed, but then we realized that a  [`merkle-path`](https://github.com/ipld/specs/blob/master/IPLD.md#what-is-a-merkle-path) was a better choice.

Both the `CID`s and the `merkle-path`s are unique global identifiers. But the `merkle-path` allows to point to mutable content (if referencing to an [IPNS](http://127.0.0.1:8080/ipns/docs.ipfs.io/guides/concepts/ipns/) link)
Plus a `CID` can be represented as `merkle-path` as well.

This also allows us to not have to dereference the `merkle-path` in order to obtain the `CID`.

### Pointing to anything [todo]
Nodes vs cids...

### The data structure
Considering all the above, a representation of a `node` as an `IPLD` object looks like this
_TODO: The links are `CIDs` and should be `merkle-paths`._

```json
{
    "origin": {
        "link": {
            "/": "QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k"
        }
    },
    "relations": [
        {
            "target": {
                "link": {
                    "/": "zdpuAvYJaZxBjTV4WH3irwThm5t2a7yTccoN9cWpDmtV4CiNz"
                }
            },
            "type": {
                "link": {
                    "/": "zdpuAvYJaZxBjTV4WH3irwThm5t2a7yTccoN9cWpDmtV4CiNz"
                }
            }
        },
        {
            "target": {
                "link": {
                    "/": "zdpuAyvmoJWTiVrCv1aCHV5xUZ1fxUf4XLkrprKPMMFCNKfj3"
                }
            }
        }
    ]
}
```



### Render vs structure [outdated]
_This is a work in progress_

If we take a classic mindmap, the connections and nodes may have different shapes, sizes, colors... This is what the render should do. It needs to understand the relations and the nodes so they can be drawn.

It would be easy to add properties such as "color", so the render can pick it up. But unless this is something intrinsic of the node or relation, should be left out of the equation.

This is because the final goal is to be able capture and organize concepts, and not to visualize them in a specific way. We need to keep the data render agnostic. It just happen that we choose a mindmap like render to start exploring how to organize and render this type of data.


### Render format [outdated]
The render should be as dumb as possible and not have to care about how the data is gathered.
So, we should prepare it before handing it out to it.

Right now, and without much exploration, a list of `CIDs` and `node objects` seems a good aproach. This list should already have merged any duplicated node.

Assming this data set:
```json
[
    {
        "/":"<CID1>",
        "relations": [
            {
                "target": "<CID2>"
            }
        ]
    },
    {
        "/":"<CID1>",
        "relations": [
            {
                "target": "<CID3>"
            }
        ]
    },
    {
        "/":"<CID2>"
    },
]
```

Should be mapped to:

```json
{
    "<CID1>": {
        "/":"<CID1>",
        "relations": [
            {
                "target": "<CID2>"
            },
            {
                "target": "<CID3>"
            }
        ]
    },
    "<CID2>":{
        "/":"<CID2>",
    },
    "<CID3>":{
        "/":"<CID3>",
    }
}
```




## Dimensions and recursivitiy
_This is an attempt to understant the data structure from a different perspective, I'm pretty sure that there is works of others into this direction, so I would really appreciate references, or any insight to my naive aporach_


### 1D
It is a requirement for our mindmap design to be able to represent relations that our mind can naturally concieve such as a bi-directional link ( `A` ⇄ `B` ) or a [`direct graph`](https://en.wikipedia.org/wiki/Directed_graph) like connections ( `A` → `B` → `C` → `A` )

We can express relations between two `CID`s as coordinate. Where the abscissa is the origin `CID` and the ordinate is the target `CID`. (`originCID`, `targetCID`)

A bi-directional link: (`A`, `B`), (`B`, `A`)  
And the direct graph: (`A`, `B`), (`B`, `C`), (`C`, `A`)

The problem is that the `IPFS` domain is single dimensional space. The points of this space are the `CID`s (assuming no collisions), which are just numbers on a line. This is a property we inherit from its [`DAG`](https://en.wikipedia.org/wiki/Directed_graph) structure.

This means that if you add information to a content (`origin`), like a `relation`, now you have modified the hash of this content, therefore the `target` content that was pointing back to the `origin` is now pointing to the older version of it (the one without a relation)

`IPLD` and therefore a `mindmap node` are part of the `IPFS` domain, so they live on this 1D world. This prevents us from making rescursive/ciclic references like the examples above, since pointing something back modifies the content it self.

### 2D

This is not the case, the reason being, is that we encapsulate the coordinate into another object, the `mindmap node`

If we treat the `mindmap node` like it lives in a paral.lel domain we gain an extra degree of freedom.
Now we have two 1D spaces. Both domains are made out `CID`s of a [multi-hash](https://github.com/multiformats/multihash) tuple. The `mindmap domain` only contains the `mindmap node`s set.  And the `content domain` contains all the `IPFS` `CID`s except the `mindmap node` `CID`s.  

So the hash of the coordinate lives on the `mindmap node`'s set(expressed with an `n` in front), while the values of the coordinate live o the only `CID`s domain
The trick here is that the node identifier is not its `CID` but the `originCID` (different dimension) and the two parameters of the relation are from the `content domain` (expressed with the `c` in front). The 

`nX`(`cA`,`cB`) 
`nY`(`cB`,`cC`)
`nZ`(`cC`,`cA`) 

This implies that a set of nodes can't be referencing to them self as a direct-graph.

A [`node cluster`](###-node-cluster) is the set of nodes that are poining to the same `CID`.
a coordinate in this space, where the abcissa is the content that is pointing at, and the ordenate is 

_I will love to get more thoughts on this, and some help in improving the wording_


## Targeting nodes
If we consider that `origin` and the `target` are references to arbirary content. What we have is nothing else but data linked 

If the origin now points to a `node`
From my perspective, this is why 
### Relationship dimensions
...

## Terminology

Terminology is becoming a problem. We need more precise vocabulary in order to just discuss about all this. At the same time this is forcing ous to re-define concepts that had a little to broad definition.

### Node
We are working towards eliminating this word from our dictionary because it keeps  generating confusion. There are a lot of types of nodes, they almost represent the same thing, but they don't.

That said, we don't have new words yet.

The original idea we were referring to was the graphical representation of a node in a mindmap. A vertex with one or more relations


### Relation
### Origin
### Target
### Node cluster
One of the [original specs](##-original-specs) was:
> It needs to work on a global domain. This means that two different mindmaps pointing to the same concept should converge if put together

We call this convergence a `node cluster` (_would love a better name_). In other words is the set of nodes that are poining to the same `CID`.

## Log (just to give a vague idea of the progress)
- `13/09/2018`: We've figured out a basic data structure to start. Defined in the section above
- `18/09/2018`: We started exploring a first render: [ipld-mindmap-pts-render](https://github.com/arxiu/ipld-mindmap-pts-render)
- `21/09/2018`: Documenting node identification. Documenting render format.
- `26/09/2018`: Render shows basic nodes with mock data, nodes are selectable and can be navigated with arrow keys
- `27/09/2018`: Converting this repo into a React-Create-App and the [ipld-mindmap-pts-render](https://github.com/arxiu/ipld-mindmap-pts-render) into a standalone component.
- `30/09/2018` : _Initital deadline reached_ (it's ok)
- `01/09/2018`: A lot of discussions regarding the way nodes are organized
- . Hierachical vs lists...
- `05/10/2018`: Render has been refactored to fit all the new find-outs. Data structures are pretty solid. A lot of UI work is required.
- `13/10/2018`: The previous week has mostly been dedicated to add mutability to the nodes in the render. Because of it, the render codebase got a little more complex that I would like.
- `14/10/2018`: There is now a [first version](https://arxiu.github.io/ipld-mindmap-pts-render) you can play around!
- `15/10/2018`: We met with Victor and discussed deeply about the this project. We also made a choppy demo at the IPFS hands on call. 
- `17/10/2018`: We're travelling (by car) from Girona to Berlin, to be around the Web3 Summit.

## Document TODOs
- Relation types as JSON linked data
- Explain `render`
- Polish Dimensions section
- Terminology
- Spell check