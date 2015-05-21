# RSP-QL Semantics

## RSP Data model

### Time instants
The time `T` is defined as an ordered and infinite sequence of discrete *time instants* `(t_1, t_2, . . .)`, 
where <code>t_i &isin; **N**</code>.  

### Timestamped triple
A *timestamped triple* is defined a pair `(d,t)`, where `d` is an RDF triple and <code>t &isin; T </code> is a time instant.

### Timestamped graph
A *timestamped graph* is defined as a pair `(g,t)`, where `g` is an RDF graph and <code>t &isin; T</code> is a time instant. 
A timestamped graph `(g,t)` can be represented as a set of timestamped triples of the form `(d_j,t)` where <code>d_j &isin; g</code>.

### RDF Stream
An *RDF stream* `S` is an unbounded sequence of timestamped triples in non-decreasing time order: 

`S = ((d_1, t_1), ... ,(d_i, t_i), ...)` where <code>&forall;i, (d_i, t_i)</code> is a timestamped RDF statement.

An *RDF stream* can also be represented as a sequence of timestamped graphs in non-decreasing time order:
`S= ((g_1,t_1), ... , (g_i,t_i), ... )`. 

### Time-varying graph
A time-varying graph `G` is a function that relates time instants <code>t &isin; T</code>to RDF graphs:
`G : T --> {g | g is an RDF graph}`
Then, an *instantaneous graph* `G(t)` is an RDF graph resulting by applying `G` to a given time instant `t`.

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
