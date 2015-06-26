# RSP-QL Semantics

## RSP Data model

### Time instants
Time `T` is defined as an ordered and infinite sequence of countable *time instants* `(t_1, t_2, . . .)`, 
where <code>t_i &isin; **T**</code>.  

In practice we could use ```xsd:dateTime``` for the time instants.

### Timestamped graph
#### Definition
A *timestamped graph* is defined as an RDF Dataset under the RDF Dataset semantics that [each graph defines its own context](http://www.w3.org/TR/2014/NOTE-rdf11-datasets-20140225/#each-named-graph-defines-its-own-context) with the following constraints.

1. There is a single named graph pair (n, g) in the RDF Dataset (where `g` is an [RDF graph](http://www.w3.org/TR/rdf11-concepts/#section-rdf-graph), and `n` is an IRI or blank node).
2. There is a single triple in the default graph of the RDF Dataset, and it has the form (n, p, t), where `n` is defined in the previous item, `p` is a predicate that captures the relationship between the time instant `t` and the graph g.

Limitations of the definition:
* does **not** cover the case when the timestamp is an interval.
* does **not** allow the default graph of a timestamped graph to have more than triple.

#### Serialization
The definition above describes an **abstract** data model for a timestamped graph because it is based on an RDF Dataset, which is an abstract entity. The serialization of such timestamped graphs is an additional issue. A timestamped graph *could* be serialized in an existing format for RDF Datasets, e.g. [Trig](http://www.w3.org/TR/2014/REC-trig-20140225/). However, the Trig syntax does not provide abbreviations for the special cases that arise for a timestamped graph. Therefore, we must introduce a new concrete syntax for timestamped graphs. There are several syntactic categories.
####  'n' is a blank node. 
The intention is that this blank node will not be shared with any other RDF graphs or RDF Datasets. Therefore, it can be represented implicitly, meaning processors will generate a new blank node for each timestamped graph they receive. As an illustration, the serialization could take the form of a space-separated tuple surrounded by parentheses: (g p t). For example, a serialization of a timestamped graph containing two triples would appear as follows
```
(
  {
       _:b foaf:name "Alice" .
       _:b foaf:mbox <mailto:alice@work.example.org> .
  }
  <http://streamreasoning.org/timeVocab#startsAtTime>
  "2012-11-02T19:14:23+01:00"^^xsd:dateTime
)
```
When we say "a timestamped graph contains two triples", that is shorthand for "a timestamped graph whose named graph contains two triples"
This syntax could potentially be abbreviated further by making some assumptions, e.g.
* the predicate is always an absolute IRI
* the timestamp is by default of datatype xsd:dateTime
With these assumptions, most of the delimiting characters become unnecessary, and the timestamped graph could be serialized so:
```
(
  {
       _:b foaf:name "Alice" .
       _:b foaf:mbox <mailto:alice@work.example.org> .
  }
  http://streamreasoning.org/timeVocab#startsAtTime
  2012-11-02T19:14:23+01:00
)  
```
#### 'n' is an IRI and the named graph is included explicitly. 
As an illustration, the serialization could take the form of a space-separated qudruple surrounded by parentheses: (n g p t). For example:
```
(
 <http://example.org/alice>
  {
       _:b foaf:name "Alice" .
       _:b foaf:mbox <mailto:alice@work.example.org> .
  }
  <http://streamreasoning.org/timeVocab#startsAtTime>
  "2012-11-02T19:14:23+01:00"^^xsd:dateTime
)
```
This could potentially be abbreviated further by making some assumptions in addition to those from the previous case, e.g.
* the name of the named graph is always an absolute IRI
With these assumptions, most of the delimiting characters become unnecessary, and the timestamped graph could be serialized so:
```
(
http://example.org/alice
  {
       _:b foaf:name "Alice" .
       _:b foaf:mbox <mailto:alice@work.example.org> .
  }
  http://streamreasoning.org/timeVocab#startsAtTime
  2012-11-02T19:14:23+01:00
)  
```

####  'n' is an IRI and the named graph is obtained by dereferencing that IRI. 
As an illustration, the serialization could take the form of a space-separated tuple surrounded by parentheses: (n p t). For example:
```
(
 <http://example.org/alice>
 <http://streamreasoning.org/timeVocab#startsAtTime>
 "2012-11-02T19:14:23+01:00"^^xsd:dateTime
)
```
This could potentially be abbreviated further by making the assumptions as in the previous cases.
With these assumptions, most of the delimiting characters become unnecessary, and the timestamped graph could be serialized so:
```
(
http://example.org/alice
http://streamreasoning.org/timeVocab#startsAtTime
2012-11-02T19:14:23+01:00
)  
```
Although there may be further abbreviations possible for these special cases, it is also desirable to be able to extend this syntax to additional cases. For example, consider situations where
* the timestamp is not in the xsd:dateTime datatype. In this case the full form for a literal could be used, "2012-11-02"^^xsd:date
* the timestamp is complex, e.g. an interval. In this case, a [nested unlabelled blank node](http://www.w3.org/TR/2014/REC-turtle-20140225/#unlabeled-bnodes) could be used.
* the same named graph is referenced by more than one timestamping triple, with timestamps corresponding to different events. In this case a larger tuple, e.g. (g p1 t1 p2 t2) could be used.

Further abbreviations may be considered after a broader range of extensions has been examined.

Note that each of the concrete examples above describes an RDF Dataset - there is a uniform abstract data model for timestamped graphs, independent of the form taken in the concrete syntax.

> **Note:** The remainder of this page may not be consistent with the definitions above.

A single triple can be streamed using a singleton graph where the timestamp is captured on the graph and the data in the triple in the graph.

There can be multiple timestamps associated with a graph `g`, e.g. a start time and an end time, or a generated time and a system processing time. The predicate `p` should be drawn from a community agreed vocabulary ([Issue 10](https://github.com/streamreasoning/RSP-QL/issues/10)).

There can be multiple graphs with the same timestamp.

### Stream
A *stream* `S` consists of a sequence of timestamped graphs `(g,p,t)`.

### Substream
A *substream* (window) `S'` of a stream `S` is a finite subsequence of `S`.

### Window functions

> Note: Window operator is reserved for later use to return time-varying graphs. Window functions work on a time instant.

A *window function* `w_\iota` of type `\iota` takes as input a stream `S`, a time instant `t`, called the reference time point, and a vector of window parameters `x` for type `\iota` and returns a substream `S'` of `S`.

The most common types of windows in practice are time- and count-based. We associate them with the window functions `w_\time`, `w_\count`, respectively. They take a fixed size ranging back in time from a reference time point `t`. Intuitively, these functions work as follows.

#### Time-based window functions

`x = (l,d)`, where `l ∈ N ∪ {∞}` and `d ∈ N`. The function `w_time(S,t,x)` returns the substream of S that contains all timestamped graphs of the last `l` time units relative to a pivot time point `t'` derived from `t` and the step size `d` (Todo: MD: a figure to illustrate). We use `l = ∞` to take all previous timestamped graphs.

#### Count-based window functions

`x = (l)`, where `l ∈ N`. The function `w_count(S,t,x)` selects a substream `S_1` of `S` based on the time instant `t'` satisfying that:
* for every `t' < t'' \leq t`, there are fewer than `l` timestamped graphs in `S` from `t''` to `t`,
* there are at least `l` timestamped graphs in `S` from `t'` to `t`.

Elements from `S_1` are those `(g_i,p_i,t_i)` from `S` having `t_i \geq t'`. In case there are more than `l` timestamped graphs in `S` from `t'` to `t`, only timestamped graphs at `t'` are removed at random.
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


