# RSP-QL Semantics

## RDF Streams

### Time instants
The time T is defined as an ordered and infinite sequence of discrete *time instants* (t_1, t_2, . . .), 
where t_i &isin; **N**.  

### Timestamped triple
A *timestamped triple* is defined a pair (d, t), where d is an RDF triple and t &isin; T is a time instant.

### Timestamped graph
A *timestamped graph* is defined as a pair (g, t), where g is an RDF graph and t &isin; T is a time instant. 
A timestamped graph (g,t) can be represented as a set of timestamped triples of the form (d_j,t) where d_j &isin; g.

### RDF Stream
An *RDF stream* S is an unbounded sequence of timestamped triples in non-decreasing time order: 
S = ((d_1, t_1), ... ,(d_i, t_i), ...) where &forall;i, (d_i, t_i) is a timestamped RDF statement.
An *RDF stream* can also be represented as a sequence of timestamped graphs in non-decreasing time order:
S= ((g_1,t_1), ... , (g_i,t_i), ... ). 



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
