# RSP-QL Semantics

## RSP Data model

### Time instants
Time `T` is defined as an ordered and infinite sequence of countable *time instants* `(t_1, t_2, . . .)`, 
where <code>t_i &isin; **T**</code>.  

In practice we could use xsd:dateTime for the time instants.

### Timestamped graph
A *timestamped graph* is defined as a triple `(g,p,t)`, where `g` is an RDF graph, <code>t &isin; T</code> is a time instant, and `p` is a predicate that captures the relationship between the time instant `t` and the graph `g`. 

A single triple can be streamed using a singleton graph where the timestamp is captured on the graph and the data in the triple in the graph.

There can be multiple timestamps associated with an individual graph, e.g. a start time and an end time, or a generated time and a system processing time.

There can be multiple graphs with the same timestamp.

TODO: Identify a vocabulary of terms for the predicate `p`.

### Stream
A stream `S` consists of a sequence of timestamped graphs `(g,p,t)`.

TODO: Define what we mean by sequence or is this a set?

### Window (data)
A window `w_S` is a set of triples from the stream S. Time-based windows are defined by an opening (`o`) and closing (`c`) time instants:

<code>w_S = {d | (d,t) &isin; S and t &isin; (o,c]}</code>


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
