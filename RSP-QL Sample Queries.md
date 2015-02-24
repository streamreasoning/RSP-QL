#RSP-QL Example Queries

This query set is based on the data of the DEBS challenge 2015 (see [DEBS Challenge 2015 data] (https://github.com/streamreasoning/RSP-QL/blob/master/DEBS%20Challenge%202015%20data.md ) )

## Data

We have not agreed on a simple schema and structure of the streams (how many streams, and the contents of each one. For each contributing query please explain your data assumptions.	  

## Queries

### Query 1

This query simply shows the use of a sliding window in RSP-QL. It intends to get the number of taxi rides that exceeded 2 miles in the last hour.

This is a simple example of the structure of a portion of the data as an RDF stream. The stream (http://debs2015.org/streams/rides) outputs a graph per ride when the user is dropped off. E.g. :

```
@prefix debs: <http://debs2015.org/onto#>

{
:ride debs:byTaxi :taxi;
	  debs:pickupAt "2013-01-01 00:00:00";
	  debs:dropoffAt "2013-01-01 00:02:00";
	  debs:distance 3. }

{	  
:ride1 debs:byTaxi :taxi1;
	  debs:pickupAt "2013-01-01 00:00:00";
	  debs:dropoffAt "2013-01-01 00:03:00";
	  debs:distance 2. }
```	

The query:

```
PREFIX debs: <http://debs2015.org/onto#>
prefix s: <http://debs2015.org/streams/>

SELECT COUNT(?ride) as ?rideCount 
FROM NAMED WINDOW :wind ON s:rides [RANGE PT1H STEP PT1H]
WHERE {
  WINDOW :win {
    ?ride debs:distance ?distance
    FILTER(?distance>2)
}}
```