# RSP-QL Semantics

## RSP Data model

### Time instants
Time `T` is defined as an ordered and infinite sequence of countable *time instants* `(t_1, t_2, . . .)`, 
where <code>t_i &isin; **T**</code>.  

In practice we could use ```xsd:dateTime``` for the time instants.

### Timestamped graph
A *timestamped graph* is defined as a triple `(g,p,t)`, where `g` is an [RDF graph](http://www.w3.org/TR/rdf11-concepts/#section-rdf-graph) interpreted under the semantics that [each graph defines its own context](http://www.w3.org/TR/2014/NOTE-rdf11-datasets-20140225/#each-named-graph-defines-its-own-context), <code>t &isin; T</code> is a time instant, and `p` is a predicate that captures the relationship between the time instant `t` and the graph `g`. 

A single triple can be streamed using a singleton graph where the timestamp is captured on the graph and the data in the triple in the graph.

There can be multiple timestamps associated with a graph `g`, e.g. a start time and an end time, or a generated time and a system processing time. The predicate `p` should be drawn from a community agreed vocabulary ([Issue 10](https://github.com/streamreasoning/RSP-QL/issues/10)).

There can be multiple graphs with the same timestamp.

### Stream
A *stream* `S` consists of a sequence of timestamped graphs `(g,p,t)`.

### Bounded Substream
See [Issue 11](https://github.com/streamreasoning/RSP-QL/issues/11).

A *bounded substream* `S_1` of a stream `S` is a sequence of timestamped graphs such that a timestamped graph `(g_i,p_i,t_i)` is in `S_1` if and only it it appears between the lower and upper bound declared in the bounded substream `S_1`.

#### Time-bounded Substream
A *time-bounded substream* is defined by two time instances providing a lower bound `t_l` and an upper bound `t_u` where `t_l <= t_u`. A timestamped graph `(g_i,p_i,t_i)` is in the time-bounded substream if and only if `t_l <= t_i <= t_u`.

#### Count-bounded Substream
A *count-bounded substream* is defined by a time instance `t` and an integer value `n` that represents the number of timestamped graphs to include in the count-bounded substream. The count-bounded substream consists of the `n` timestamped graphs at or before time instance `t`. That is, a timestamped graph `(g_i,p_i,t_i)` is in the count-bounded substream if and only if there are less than or equal to `n` timestamped graphs between it and the time instance `t`.

### Stream Snapshot
A *stream snapshot* consists of the union of all triples in a bounded substream.


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
