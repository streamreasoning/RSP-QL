#RSP-QL Example Queries

This query set is based on the data of the DEBS challenge 2015 (see [DEBS Challenge 2015 data] (https://github.com/streamreasoning/RSP-QL/blob/master/DEBS%20Challenge%202015%20data.md ) )

## Data

We have not agreed on a simple schema and structure of the streams (how many streams, and the contents of each one. For each contributing query please explain your data assumptions.	  

## Queries

### Query 1

(added by jpc)

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

### Query 2
(by Alessandra)

The query sketch generates a graph whenever more than 20 taxis have the same drop off location within 30 minutes. 

The input stream contains fares and trip data described as in http://chriswhong.com/open-data/foil_nyc_taxi/
NOTE: I thought I needed two windows to do that but I think that is not necessary. Would be interesting to change it to get the number of taxis that are arriving at a location and leaving that location within 30 min (e.g. to spot events or things happening or other patterns). I have used drop off location as the exact location, probably a notion of distance (e.g. within distance 3 from that location) would make more sense.

```
PREFIX debs: <http://debs2015.org/pred#>
PREFIX s: <http://debs2015.org/streams/>

REGISTER STREAM :query AS

SELECT ?location (count(distinct ?taxi) as ?taxinumber
FROM NAMED WINDOW :w1 ON s [RANGE PT1h step PT30H]
WHERE {
 ?location a :dropoffLocation
 WINDOW :w1 {
 ?taxi pred:dropoff ?location 
 }
 GROUPBY ?location
 HAVING (?taxinumber >= 20)
}
```

### Query 3

(by Bernhard)

The data set that is used as a "RDF"ed version of http://www.debs2015.org/call-grand-challenge.html
Temporal data is encoded in the W3C's Time Ontology and spatial data in OpenGIS' ontology.

"Uber Ride of Glory Query"

```
PREFIX debs: <http://debs2015.org/onto#>
prefix tstream: <http://debs2015.org/streams/>
PREFIX time: <http://www.w3.org/2006/time#>
PREFIX geodata: <http://linkedgeodata.org/ontology/addr%3A>
PREFIX geo: <http://www.opengis.net/ont/geosparql#>
PREFIX geof: <http://www.opengis.net/def/geosparql/function/>
PREFIX wgs84: <http://www.w3.org/2003/01/geo/wgs84_pos#>

SELECT ?time ?district
FROM NAMED WINDOW :wind ON tstream:rides [RANGE PT6H STEP PT6H]
WHERE {
 WINDOW :drop {
				#Dropoff
				?ride debs:dropoff_latitude ?lat;
					  debs:dropoff_longitude ?lng;
					  debs:dropoff_datetime ?time.
					?time time:hour ?drop_hour.
					?feature geo:hasGeometry ?dropGeom;
						   wgs84:lat ?lat;
						   wgs84:lng ?lng;
					   geodata:district ?district.
				FILTER(?drop_hour < 4)
				FILTER(22 < ?drop_hour)
			} 
 Window :pick {
				#Pickup
				?ride debs:pickup_latitude ?lat;
					  debs:pickup_longitude ?lng;
					  debs:pickup_datetime ?time.
					?time time:hour ?pick_hour.
					?place geo:hasGeometry ?pickGeom;
						   wgs84:lat ?lat;
						   wgs84:lng ?lng;
					   geodata:district ?district.
				FILTER(?pick_hour < 4)
				FILTER(22 < ?pick_hour)
			}
		#RoG drop off location has to differ max 0.1 from the first
		# has to differ at least for one hour
		FILTER (?pick_hour - ?drop_hour > 1)
		FILTER (geof:distance(?dropGeom, ?pickGeom, units:mile, 0.1))
}
```


### Query 4
(by Bernhard)

Most profitable Trips

```
PREFIX debs: <http://debs2015.org/onto#>
prefix tstream: <http://debs2015.org/streams/>

SELECT ?distance (?amount - ?tax -?tips -?tolls) AS ?profit
FROM NAMED WINDOW :wind ON tstream:rides [RANGE PT1H STEP PT1H]
WHERE {
  WINDOW :profit {
	?ride debs:trip_distance ?distance;
		  debs:total_amount ?amount;
		  debs:mta_tax ?tax;
		  debs:tip_amount ?tips;
		  debs:tolls_amount ?tolls.
  }
}
```

### Query 5
(by Fariz)

Lucky taxi rides (taxi rides that did not meet any red traffic light
-- always get the green ones) in the last 1 hour
 
Feature: Negation using NOT EXISTS
 
Data:
 
``` 
@prefix debs: <http://debs2015.org/onto#> .
@prefix ex: <http://example.org/> .
 
{
:ride1 debs:byTaxi :taxi1;
debs:pickupAt "2013-01-01 00:00:00";
debs:dropoffAt "2013-01-01 00:02:00";
debs:distance 3;
ex:stoppedAt :trafficLight1, :trafficLight2 }
 
# :ride2 didn't meet any red traffic light
{
:ride2 debs:byTaxi :taxi2;
debs:pickupAt "2013-01-01 00:00:00";
debs:dropoffAt "2013-01-01 00:03:00";
debs:distance 2 }
``` 
 
Query:
 
```
PREFIX debs: <http://debs2015.org/onto#>
PREFIX s: <http://debs2015.org/streams/>
PREFIX ex: <http://example.org/>
 
SELECT ?luckyRide
FROM NAMED WINDOW :win ON s:rides [RANGE PT1H STEP PT1H]
WHERE {
  WINDOW :win {
    ?luckyRide debs:byTaxi ?taxi .
    FILTER NOT EXISTS {?luckyRide ex:stoppedAt ?trafficLight}
}}
```
