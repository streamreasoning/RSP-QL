## Serialization
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
