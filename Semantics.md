# RSP-QL Semantics

## RSP Data model

### Temporal entities
Time `T` is defined as an ordered and infinite sequence of countable *temporal entities* `(t_1, t_2, . . .)`, 
where <code>t_i &isin; **T**</code>. Temporal entities can be referred to as timestamps.

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

> Example of timestamped graph needed here

### Stream
A *stream* `S` consists of a sequence of timestamped graphs.

> **Discussion**: Given a certain `p`, elements on the stream having that predicate are ordered according to `t`. If the stream contains elements s1 and s2 associated with the same predicate, with s1 preceding s2, then it should not be the case that the timestamp of s1 is greater than the timestamp of s2.

**Observation:** There can be multiple graphs with the same timestamp in the stream.
> It has been pointed out that this statement might be problematic, as graphs cold no longer be used for punctuation purposes. Comparatively, we have not found a constraint on this in similar models e.g. CQL: *there could be zero, one, or multiple elements with the same timestamp in a stream*. To verify.

> Example of a stream of timestamped graph needed here
> Example (by Robin) needs update
> A sensor registers vehicles at a street crossing. At a defined time interval all observations made since the last output is pushed to the stream, e.g. output at time t5 is:
```
(g_1,"generated at",t_5)
(g_2,"generated at",t_5)
(g_3,"generated at",t_5)
(g_1,"observed at",t_1)
(g_2,"observed at",t_2)
(g_3,"observed at",t_3)
```


### Substream
A *substream* (also known as window) `S'` of a stream `S` is a finite subsequence of `S`.

### Window functions

> Note: Window operator is reserved for later use to return time-varying graphs. Window functions work on a time instant. 

> Note: In the following, we take the case where temporal entities in the stream are time instants.

A *window function* <code>w<sub>&iota;</sub></code> of type <code>&iota;</code> takes as input a stream `S`, a time instant `t`, called the reference time point, and a vector of window parameters `x` for type <code>&iota;</code> and returns a substream `S'` of `S`.

The most common types of windows in practice are time-based and count-based. We associate them with the window functions <code>w<sub>&tau;</sub></code>, <code>w<sub>count</sub></code>, respectively. They take a fixed size ranging back in time from a reference time point `t`. These functions work as follows.

#### Time-based window functions

`x = (l,d)`, where `l ∈ N ∪ {∞}` and `d ∈ N`. The function `w_time(S,t,x)` returns the substream of S that contains all timestamped graphs of the last `l` time units relative to a pivot time point `t'` derived from `t` and the step size `d` (Todo: MD: a figure to illustrate). We use `l = ∞` to take all previous timestamped graphs.

Formally:

`w_\time(S,t,(l,d)) = {(g,p,t_1)\in S \mid t'-l < t_1 \leq t'}`, 

where `t'= \lfloor \frac{t}{d} \rfloor \cdot d`

#### Count-based window functions

`x = (l)`, where `l ∈ N`. The function `w_count(S,t,x)` selects a substream `S_1` of `S` based on the time instant `t'` satisfying that:
* for every `t' < t'' \leq t`, there are fewer than `l` timestamped graphs in `S` from `t''` to `t`,
* there are at least `l` timestamped graphs in `S` from `t'` to `t`.

Elements from `S_1` are those `(g_i,p_i,t_i)` from `S` having `t_i \geq t'`. In case there are more than `l` timestamped graphs in `S` from `t'` to `t`, only timestamped graphs at `t'` are removed at random.

Formally:

`w_count(S,t,(l)) = {(g,p,t'')\in S \mid t' < t'' \leq t} \cup X(S[t',t'], l - #S[t'+1,t])`,

where

* `S[t_1,t_2] = {(g,p,t'')\in S \mid t_1 \leq t'' \leq t_2}`,
* `#S[t_1,t_2]` is the number of elements of this set,
* `t'` satisfies that `#S[t',t] \geq l` and `#S[t'+1,t] < l`.

___

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

## Operators

### Time-based Sliding Window (operator)
A time based sliding window `W` takes as input a stream and produces a time-varying graph `G_W`. The parameters of `W` are &alpha; (window length) and &beta; (window slide). The application of a window `W` over a stream is denoted as `W(S)`
For a given time t the application of the window W(S) is an instantaneous graph G_W(t) such that:

<code>G_W(t)={d | (d,t_d) &isin; S and t_d &isin; (o_p,t]}</code>

where o_p is the most recent window opening time.

## RSP-QL Definition

An *RSP-QL query* `Q` is a tuple `Q=(SE,SDS,QF)` where *SE* is an RSP-QL algebraic expression, 
*SDS* is an RSP-QL dataset and *QF* is the Query Form

### RSP-QL Dataset
An *RSP-QL dataset* **SDS** is defined as the set of: a default graph `G_0`, n named graphs ``G_i` and m named
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

