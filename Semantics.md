# RSP-QL Semantics

## RSP Data model

### Temporal entities
Time <code>**T**</code> is defined as an ordered and infinite sequence of countable *temporal entities* <code>(t<sub>1</sub>, t<sub>2</sub>,...)</code>, 
where each <code>t<sub>i</sub> &isin; **T**</code>. Temporal entities can be referred to as timestamps.

#### Instants and Intervals
Following the concepts of the Time Ontology (http://www.w3.org/TR/owl-time/), a temporal entity can be a time instant or a time interval.

### Timestamped graph
A *timestamped graph* is defined as an RDF Dataset under the RDF Dataset semantics that [each graph defines its own context](http://www.w3.org/TR/2014/NOTE-rdf11-datasets-20140225/#each-named-graph-defines-its-own-context) with the following constraints.

1. There is a single named graph pair (n, g) in the RDF Dataset (where `g` is an [RDF graph](http://www.w3.org/TR/rdf11-concepts/#section-rdf-graph), and `n` is an IRI or blank node).
2. There is a ~~single~~ triple in the default graph of the RDF Dataset, and it has the form (n, p, t), where `n` is defined in the previous item, `p` is a predicate that captures the relationship between the temporal entity `t` and the graph g.

Limitations of the definition:
* does **not** allow the default graph of a timestamped graph to have more than triple.

> **Discussion:** There could be multiple timestamps associated with a graph `g`, e.g. a start time and an end time, or a generated time and a system processing time. The predicate `p` should be drawn from a community agreed vocabulary ([Issue 10](https://github.com/streamreasoning/RSP-QL/issues/10)).

> **Discussion:** More than one triple may be necessary to represent the time metadata for each graph.

#### Example: 
The following timestamped graph `:g1` contains 2 triples that state that Darko and Axel are in the Red Room. The `p` predicate used in this example is the PROV ``prov:generatedAtTime`. In this example the named graph `:g1` contains the data contents (triples). The format in the example follows [TriG](http://www.w3.org/TR/trig/), although does not imply any specific serialization or formatting, it simply shows the data structured according to the RDF stream model. Prefixes (e.g. `prov:`) are used for readability.

```
:g1 {:axel :isIn :RedRoom. :darko :isIn :RedRoom} {:g1,prov:generatedAtTime,"2001-10-26T21:32:52"}
```


### RDF Stream
A *RDF stream* `S` consists of a sequence of timestamped graphs whose elements sharing the same predicate are ordered by the partial order associated with this predicate on the timestamps. 

**Order:** The partial order must respect the natural order of time. In particular, if every time instant within the closure of temporal entity `X` is earlier than every time instant within the closure of temporal entity `Y`, then `X <= Y` (where closure of a time instant `t` is defined as the degenerate interval `[t, t]`, and closure of an interval is defined in the usual way) 

Furthermore, the usual mathematical requirements of a partial order apply:

- a) Reflexivity `X <= X`
- b) Antisymmetry `X <= Y` and `Y <= X` implies `X = Y`
- c) Transitivity `X <= Y` and `Y <= Z` implies `X <= Z`. 

On the following we may refer to RDF stream simply as stream.

#### Example: 
A stream produces data that indicates where a person is at a given time. The `p` predicate used in this example is the PROV ``prov:generatedAtTime`. In this example the named graphs (`:g1`,`:g2`, etc.) contain the streaming data contents (for brevity the contents are represented by the dots `...`). 

```
:g1 {...}{:g1,prov:generatedAtTime,t1}
:g2 {...}{:g2,prov:generatedAtTime,t2}
:g3 {...}{:g3,prov:generatedAtTime,t3}
:g4 {...}{:g4,prov:generatedAtTime,t4}
...
```

We can expand the content of each named graph, which is a set of triples:

```
:g1 {:axel :isIn :RedRoom. :darko :isIn :RedRoom} {:g1,prov:generatedAtTime,t1}
:g2 {:axel :isIn :BlueRoom. }                     {:g2,prov:generatedAtTime,t2}
:g3 {:minh :isIn :RedRoom. }                      {:g3,prov:generatedAtTime,t3}
...
```

> **Discussion**: Given a certain `p`, elements on the stream having that predicate are ordered according to `t`. If the stream contains elements s1 and s2 associated with the same predicate, with s1 preceding s2, then it should not be the case that the timestamp of s1 is greater than the timestamp of s2.

**Observation:** There can be multiple graphs with the same timestamp in the stream.
> It has been pointed out that this statement might be problematic, as graphs cold no longer be used for punctuation purposes. Comparatively, we have not found a constraint on this in similar models e.g. CQL: *there could be zero, one, or multiple elements with the same timestamp in a stream*. To verify.


### Substream
A *substream* (also known as window) `S'` of a stream `S` is a subsequence of `S`.

### Window functions

> Note: Window operator is reserved for later use to return time-varying graphs. Window functions work on a time instant. 

> Note: In the following, we take the case where temporal entities in the stream are time instants.

A *window function* <code>w<sub>&iota;</sub></code> of type <code>&iota;</code> takes as input a stream `S`, a predicate `p`, a time instant `t`, called the reference time point, and a vector of window parameters `x` for type <code>&iota;</code> and returns a substream `S'` of `S` that contains only timestamped graphs associated with `p` and timestamps valid at `t` according <code>&iota;</code> and `x`.

The most common types of windows in practice are time-based and count-based. We associate them with the window functions <code>w<sub>&tau;</sub></code>, <code>w<sub>#</sub></code>, respectively. They take a fixed size ranging back in time from a reference time point `t`. These functions work as follows.

#### Time-based window functions

`x = (l,d)`, where `l ∈ N ∪ {∞}` and `d ∈ N`. The function <code>w<sub>&tau;</sub>(S,p,t,x)</code> returns the substream of S that contains all timestamped graphs associated with predicate `p` of the last `l` time units relative to a pivot time point `t'` derived from `t` and the step size `d` (Todo: MD: a figure to illustrate). We use `l = ∞` to take all previous timestamped graphs.

Formally:

<code>w<sub>&tau;</sub>(S,p,t,(l,d)) = {((n,g),p,t<sub>1</sub>)&isin; S &mid; t'-l < t<sub>1</sub> &leq; t'}</code>, 

where <code>t'= &lfloor;t/d&rfloor; \cdot d</code>

#### Count-based window functions

`x = (l)`, where `l ∈ N`. The function <code>w<sub>#</sub>(S,p,t,x)</code> selects a substream <code>S<sub>1</sub></code> of `S` based on the time instant `t'` satisfying that:
* for every <code>t' < t'' &leq; t</code>, there are fewer than `l` timestamped graphs associated with predicate `p` in `S` from `t''` to `t`,
* there are at least `l` timestamped graphs associated with predicate `p` in `S` from `t'` to `t`.

Elements from <code>S<sub>1</sub></code> are those <code>((n<sub>i</sub>,g<sub>i</sub>),p,t<sub>i</sub>)</code> from `S` having <code>t<sub>i</sub> &geq; t'</code>. In case there are more than `l` timestamped graphs associated with predicate `p` in `S` from `t'` to `t`, only timestamped graphs at `t'` are removed at random.

Formally:

<code>w<sub>#</sub>(S,p,t,(l)) = {((n,g),p,t'') &isin; S &mid; t' < t'' &leq; t} ∪ X(S&vert;<sub>p</sub>[t',t'], l - #S&vert;<sub>p</sub>[t'+1,t])</code>,

where

* <code>S&vert;<sub>p</sub>[t<sub>1</sub>,t<sub>2</sub>] = {((n,g),p,t'') &isin; S &mid; t<sub>1</sub> &leq; t'' &leq; t<sub>2</sub>}</code>,
* <code>#S&vert;<sub>p</sub>[t<sub>1</sub>,t<sub>2</sub>]</code> is the number of elements of this set,
* `t'` satisfies that <code>#S&vert;<sub>p</sub>[t',t] &geq; l</code> and <code>#S&vert;<sub>p</sub>[t'+1,t] &lt; l</code> 

Note that a bounded substream maintains the timestamped graph contexts of the original stream.

> See [Issue 11](https://github.com/streamreasoning/RSP-QL/issues/11).

#### Time-bounded Substream
A *time-bounded substream* is defined by two time instances providing a lower bound `t_l` and an upper bound `t_u` where `t_l <= t_u`. A timestamped graph `(g_i,p_i,t_i)` is in the time-bounded substream if and only if `t_l <= t_i <= t_u`.

#### Count-bounded Substream
A *count-bounded substream* is defined by a time instance `t` and an integer value `n` that represents the number of timestamped graphs to include in the count-bounded substream. The count-bounded substream consists of the `n` timestamped graphs at or before time instance `t`. That is, a timestamped graph `(g_i,p_i,t_i)` is in the count-bounded substream if and only if there are less than or equal to `n` timestamped graphs between it and the time instance `t`.

### Stream Snapshot
A *stream snapshot* consists of the union of all triples in a bounded substream.

___

> **Note:** The remainder of this note needs to be updated to reflect the data model agreed at the ESWC RSP Workshop (presented above).

___


## RSP-QL Definition

An *RSP-QL query* `Q` is a tuple `Q = (SE,SDS,QF)` where `SE` is an RSP-QL algebraic expression, 
`SDS` is an RSP-QL dataset and `QF` is the Query Form

### RSP-QL Dataset
An *RSP-QL dataset* `SDS` is defined as the set of: a default graph `G_0`, n named graphs `G_i` and m named
time-varying graphs obtained by the application of time-based sliding windows over `p` streams:
```
SDS ={G0,
      (u1,G1), . . . , (un,Gn),
      (w1,W1(S1)), . . . , (wj ,Wj(S1)),
      (wj+1,Wj+1(S2)), . . . , (wk,Wk(S2)),
      . . .
      (wl,Wl(Sp)), . . . , (wm,Wm(Sp))}
```

Notice that different windows can be applied to the same stream `Si` in the same SDS.


#### Time-varying graphs
A *time-varying graph* `G` is a function that relates time instants `t ∈ T` to RDF graphs:

```
G : T → {g | g is an RDF graph}
```

An *instantaneous RDF graph* `G(t)` is the RDF graph identified by the time-varying graph `G` at the given time instant `t`.

## RSP-QL Continuous Evaluation

We extend the definition of SPARQL evaluation semantics to take into account the time dimension: we add a third parameter, evaluation time t, in the eval function signature.

Given an RSP-QL dataset SDS, an algebraic expression SE and an evaluation time instant t, we define

eval(SDS(G), SE, t)

as the evaluation of SE at time t with respect to the RSP-QL dataset SDS having active timevarying graph G.
This new concept requires a revision of the definitions of the existing SPARQL evaluation of algebraic operators.

## References:
* EP-SPARQL: a unified language for event processing and stream reasoning.
Anicic, D., Fodor, P., Rudolph, S., & Stojanovic, N. In WWW (p. 635-644). ACM. 2011.
* C-SPARQL: a Continuous Query Language for RDF Data Streams. 
Barbieri, D. F., Braga, D., Ceri, S., Della Valle, E., & Grossniklaus, M. Int. J. Semantic Computing, 4(1), 3-25. 2010.
* Enabling query technologies for the semantic sensor web. 
Calbimonte, J.-P., Jeung, H., Corcho, Ó., & Aberer, K. Int. J. Semantic Web Inf. Syst., 8(1), 43-63. 2012.
* RSP-QL Semantics: a Unifying Query Model to Explain Heterogeneity of RDF Stream Processing Systems. 
D. Dell’Aglio, E. Della Valle, J.-P. Calbimonte, O. Corcho. Int. J. Semantic Web Inf. Syst, 10(4). (in press). 2015.
* A Native and Adaptive Approach for Unified Processing of Linked Streams and Linked Data.
Phuoc, D. L., Dao-Tran, M., Parreira, J. X., & Hauswirth, M.In ISWC (Vol. 7031, p. 370-388). Springer. 2011.
* LARS: A Logic-based Framework for Analyzing Reasoning over Streams.
Beck, H., Dao-Tran, M., Eiter, T., Fink, M. In AAAI. 2015.
* RDF 1.1: On Semantics of RDF Datasets. Zimmerman, Antoine, ed.. 2014.  http://www.w3.org/TR/2014/NOTE-rdf11-datasets-20140225.

> this example could be integrated to the main text body
> ### Beyond time instants: intervals & more 
> Usign the previously described model, intervals can be specified for a graph in the following way: Given p1 and p2 representing start and end time predicates, then `(g,p1,t1)` and `(g,p2,t2)` denote that g is defined in an interval [t1,t2]. As an example:
> ````
:g_1, :startsAt, "2015-06-18T12:00:00"^^xsd:dateTime
:g_1, :endsAt, "2015-06-18T13:00:00"^^xsd:dateTime
```
> Or even:
```
:g_2 :validBetween
    [:startsAt "2015-06-18T12:00:00"^^xsd:dateTime;
    :endsAt "2015-06-18T13:00:00"^^xsd:dateTime]
```    

