This query continuously look for bars where people are falling in love like [http://en.wikipedia.org/wiki/Francesca_da_Rimini#In_Inferno Paolo and Francesca in Dante's Divine Comedy] because of a book by [http://en.wikipedia.org/wiki/Galehaut Gallehault]. 

The query checks :
* over the default graph containing the points of interest (POIs) of http://somesocialnetwork.org/ that the POI is a bar. 
* over the a 4 hour long window on the stream from http://someinvasivesensornetwork.org, that pairs of people entered in the poi in different moments (if they enter together they get discarted). 
* over the same stream, with a time window of 30 minutes in the past, that pairs of those people have been staying close by for at least 30 minutes. Note: that this may require some resoning being the property isCloseBy symmetric. 
* over the same stream but with a short time window of 10 minutes, that the same pairs exit together. 

As output, for each bar, it streams out an RDF graph with the list of pairs and the total number of pairs that felt in love. 

Note that this example query covers features of C-SPARQL, CQELS, and SPARQL-Stream as well as new features missing in all RSP languages:
* From C-SPARQL it takes the REGISTER clause, the FROM STREAM clause as dataset clause, the AT clause to access the timestamp (in C-SPARQL, AT is implemented with the timestamp() function) and the aggregates (which are computed in parallel without shrinking the result set, but extending it). 
* From CQELS it takes the idea of the STREAM keyword in the WHERE clause, herein defined as WINDOW. 
* From SPARQL-Stream it takes the ISTREAM clasue, that ask the RSP engine to use the R2S operator, and the notion of windows in the past. 

Differently from a previous version of this query, it no longer covers features of EP-SPARQL such as SEQ or the getDuration() function. This reflects the decision to layer the complex event processing language on a continuous querying one. 

The new features are:
* the usage of an IRI to identify the query (and its stream of results)
* the optional UNDER ENTAILMENT REGIME clause #PW: what is the main motivation for making the entailment regime explicit? basically, i like it, but am not sure, if it would/could lead to conflicts. for instance, an engine, which implements our RSP syntax, would need to be capable of the specified entailment regimes. and i am not sure, if/how this should be ensured.
* the FROM NAMED WINDOW ON STREAM <<stream iri>> <<window>> AS << window name>> clause in the dataset declaration 
* the WINDOW keyword in the WHERE clause

<tt>
 PREFIX e: <http://somevocabulary.org/> 
 PREFIX s: <http://someinvasivesensornetwork.org/streams#>
 PREFIX g: <http://somesocialnetwork.org/graphs#>
 PREFIX : <http://acrasycompany.org/rsp>
 REGISTER STREAM :GallehaultWasTheBar 
 UNDER ENTAILMENT REGIME <http://www.w3.org/ns/entailment/RIF>
 AS
 CONSTRUCT ISTREAM { 
  ?poi rdf:type :Gallehault ; 
       :count ?howmanycouples ;
       :for (?somebody ?someoneelse)   						 
 } 
 FROM NAMED WINDOW :veryLongWindow ON s:1 [RANGE PT4H STEP PT1H] 
 FROM NAMED WINDOW :longWindow ON s:1 [FROM NOW-PT35M TO NOW-PT5M STEP PT5M] 
 FROM NAMED WINDOW :shortWindow ON s:1 [RANGE PT10M STEP PT5M]
 FROM NAMED GRAPH g:SocialGraph
 FROM GRAPH g:POIs
 WHERE {
  ?poi rdf:type e:bar . 
  WINDOW :veryLongWindow {
        {?somebody e:enters ?poi} BEGIN AT ?t3
        {?someoneelse e:enters ?poi} BEGIN AT ?t4
        FILTER(?t3>?t4) 
  }
  WINDOW :longWindow {
      {
        ?somebody e:isCloseTo ?someoneelse 
        MINUS { ?somebody e:isCloseTo ?yetanotherone . FILTER (?yetanotherone != ?someoneelse) } 
      } WITH DURATION ?duration
      FILTER (?duration>="PT30M"^^xsd:duration)
  }
  WINDOW :shortWindow {
      { ?somebody e:exits ?bar} BEGIN AT ?t1
      { ?someoneelse e:exits ?bar } BEGIN AT ?t2 
      FILTER (abs(?t2-?t1)<"PT1M"^^xsd:duration )
  }
  GRAPH g:SocialGraph { 
      FILTER NOT EXIST { ?somebody e:knows ?someoneelse }
  }
  FILTER (?somebody != ?someoneelse)
 }
 AGGREGATE {
  GROUP BY ?poi 
  COUNT(?somebody) AS ?howmanycouples 
 }
</tt>
