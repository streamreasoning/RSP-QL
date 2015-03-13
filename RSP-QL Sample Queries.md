#RSP-QL Example Queries

This query set is based on the data of the DEBS challenge 2015 (see [DEBS Challenge 2015 data] (https://github.com/streamreasoning/RSP-QL/blob/master/DEBS%20Challenge%202015%20data.md ) )

## Data

We have not agreed on a simple schema and structure of the streams (how many streams, and the contents of each one. For each contributing query please explain your data assumptions.	  

## Queries

### Query 1

(added by jpc)

This query simply shows the use of a sliding window in RSP-QL. It intends to get the number of taxi rides that exceeded 2 miles in the last hour.

This is a simple example of the structure of a portion of the data as an RDF stream. The stream (http://debs2015.org/streams/trips) outputs a graph per ride when the user is dropped off. E.g. :

```
@prefix debs: <http://debs2015.org/onto#> .
@prefix prov: <http://www.w3.org/ns/prov#> .
@prefix wgs84: <http://www.w3.org/2003/01/geo/wgs84_pos#> .

:t1 {
 :pickup1 prov:startedAtTime "2013-01-01T00:00:00"^^xsd:dateTime;        
          prov:atLocation [wgs84:lat 40.716976; wgs84:long -73.956528].
 :dropoff1 prov:startedAtTime "2013-01-01T00:10:00"^^xsd:dateTime;
		   prov:atLocation [wgs84:lat 41.716976; wgs84:long -73.956528].		   		   		   
 :trip1  debs:dropoff :dropoff1;
         debs:pickup  :pickup1;
		 debs:taxi :taxi25;
         debs:duration 35;		   
         debs:distance 1.5;
     	 debs:paymentType "CSH";
         debs:fare 3.50;
         debs:surcharge 0.50;
		 debs:tax 0.50;
		 debs:tip 0.00;
		 debs:tolls 0.00.
}

```	

The query:

```
PREFIX debs: <http://debs2015.org/onto#>
prefix s: <http://debs2015.org/streams/>

SELECT COUNT(?ride) as ?rideCount 
FROM NAMED WINDOW :wind ON s:trips [RANGE PT1H STEP PT1H]
WHERE {
  WINDOW :win {
    ?ride debs:distance ?distance
    FILTER(?distance>2)
}}
```

The data about taxis can be in a different stored  graph:
http://debs2015.org/data/taxis

```
@prefix debs: <http://debs2015.org/onto#> .

:taxi3 debs:medallion "07290D3599E7A0D62097A346EFCC1FB5";
       debs:license "E7750A37CAB07D0DFF0AF7E3573AC141".
```	   	   

About data units. they are not included in the data stream (although they could) but we could also specify them through class restrictions:

```
@prefix unit:<http://purl.oclc.org/NET/muo/ucum/unit>.

:Trip rdf:type owl:Class;
      rdfs:subClassOf [ rdf:type owl:Restriction ;
                        owl:onProperty :distanceUnit ;
                        owl:someValuesFrom [ rdf:type owl:Class ;
                                             owl:oneOf (unit:meter)] 

```

### Query 2
(by Alessandra)


The input stream contains fares and trip data described as in http://chriswhong.com/open-data/foil_nyc_taxi/

The first query generates a graph whenever more than 20 taxis have the same drop off location within 30 minutes. I thought I needed two windows to do that but I think that is not necessary. Would be interesting to change it to get the number of taxis that are arriving at a location and leaving that location within 30 min (e.g. to spot events or things happening or other patterns). I have used drop off location as the exact location, probably a notion of distance (e.g. within distance 3 from that location) would make more sense.

```
PREFIX debs: <http://debs2015.org/pred#>
PREFIX s: <http://debs2015.org/streams/>

REGISTER STREAM :query AS

SELECT ?location (count(distinct ?taxi) as ?taxinumber
FROM NAMED WINDOW :w1 ON s [RANGE PT1h step PT30M]
WHERE {
 ?location a :dropoffLocation.
 WINDOW :w1 {
 ?taxi debs:dropoff ?location.
 }
 GROUPBY ?location
 HAVING (?taxinumber >= 20)
}
```

### Query 3
(by Alessandra)

The second query is a variation of the second query of the DEBS challenge. It uses subquery to generate the top 3 profitable pickup locations in the last 30 minutes.
The highest profit is defined as the total amount of fares collected at a pick_up location in the last 30 minutes. I am not sure about subqueries like that, but aggregates in the WHERE of a CONSTRUCT wouldn't work I believe? 

```
PREFIX debs: <http://debs2015.org/pred#>
PREFIX s: <http://debs2015.org/streams/>

REGISTER STREAM :query AS

CONSTRUCT ISTREAM 
{
    ?location debs:profit ?totalamount
}

FROM NAMED WINDOW :w1 ON s [RANGE PT30M step PT15M]

WHERE 
{
 { SELECT (SUM(?amount) AS ?totalamount) ?location
  {
  WINDOW :w1 
  {
   ?taxi debs:pickup ?location
   ?location debs:amount ?amount
  }
 
   GROUP BY ?location
   ORDER BY desc(?totalamount)
   LIMIT 3
 
 }
}
```

### Query 4

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


### Query 5
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

### Query 6
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


### Query 7
(by Peter)

In which neighborhoods did pickups increase in the last hour compared to the previous hour?
In other words: Where did demand for taxis rise in the last hour? Combining with static knowledge from geonames about neighbourhoods.

Alternatives:
In which neighborhoods did pickups decrease in the last hour compared to the previous hour?
In which neighborhoods did dropoffs increase in the last hour compared to the previous hour?
In which neighborhoods did dropoffss decrease in the last hour compared to the previous hour?

Query:
 
```
PREFIX debs: <http://debs2015.org/onto#>
PREFIX s: <http://debs2015.org/streams/>
PREFIX gn: <http://www.geonames.org/ontology#>
PREFIX ex: <http://example.org/>
PREFIX geof: <http://www.opengis.net/def/geosparql/function/>
 
SELECT ?neighbourhood ( COUNT(?newPickups) / COUNT(?oldPickups) AS ?increase )
FROM NAMED WINDOW :newPickups ON s:rides [RANGE PT1H STEP PT1H]
FROM NAMED WINDOW :oldPickups ON s:rides [FROM NOW-PT2H TO NOW-PT1H STEP PT1H]
FROM GRAPH gn:geonames

WHERE {
		?oldPickups gn:neighbourhood ?neighbourhood 
		?newPickups gn:neighbourhood ?neighbourhood 
	WINDOW :newPickups {
		?newPickups debs:pickup_latitude ?nlat
		?newPickups debs:pickup_longitude ?nlon
}
	WINDOW :oldPickups {
		?oldPickups debs:pickup_latitude ?olat
		?oldPickups debs:pickup_longitude ?olon
}
FILTER ( ?increase >= 1.2 ) #increase by 20%

}
```

Other query examples (to be formulated):
* Find profitable spots to drive to (combination with event data): Where is an event (ending) right now (get public event data) and where are not many pickups in the last 30min?
* Merge taxi complaints (https://nycopendata.socrata.com/Social-Services/311-Service-Requests-from-2010-to-Present/erm2-nwe9) with trips: in which neighborhoods have there been most complaints divided by trips (complaint rate per trip) in the last hour?
* Event detection: Many (increased) outgoing trips at same area in last 30min
* Most efficient taxi driver = fastest pickup after dropoff in the last hour ~ shortest time empty
* Least efficient taxi driver = slowest pickup after dropoff in the last hour ~ longest time empty
* Empty taxis in a certain radius: how many taxis had a dropoff in the last 10 mins around my location? (option: dropoff, but not a new pickup)
* Empty times: at which locations/cells did the longest time pass after dropoffs until a new pickup for the same taxi occurred?
* What are the most used routes between neighborhoods? i.e. taxi routes from district a to district b?
* Detect traffic congestion: comparing 2 windows, speed of trips in area has decreased...
* Where does most tipping occur?
* Where are most taxis empty currently?
* Patterns? can we capture if one user goes from A to B, then after some time from B to C, etc.?
* Can we combine taxi availability data with real-time public transport data to determine whether taking a taxi or a public bus for a given ride right now?
