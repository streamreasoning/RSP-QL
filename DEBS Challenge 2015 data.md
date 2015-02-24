DEBS 2015 Grand Challenge example data: http://www.debs2015.org/call-grand-challenge.html
======

Data source : http://chriswhong.com/open-data/foil_nyc_taxi/


##Data

Provided data consists of reports of taxi trips including starting point, drop-off point, corresponding timestamps, and information related to the payment. Data are reported at the end of the trip, i.e., upon arrive in the order of the drop-off timestamps.
The specific attributes are listed below:

 * medallion	an md5sum of the identifier of the taxi - vehicle bound
 * hack_license	an md5sum of the identifier for the taxi license
 * pickup_datetime	time when the passenger(s) were picked up
 * dropoff_datetime	time when the passenger(s) were dropped off
 * trip_time_in_secs	duration of the trip
 * trip_distance	trip distance in miles
 * pickup_longitude	longitude coordinate of the pickup location
 * pickup_latitude	latitude coordinate of the pickup location
 * dropoff_longitude	longitude coordinate of the drop-off location
 * dropoff_latitude	latitude coordinate of the drop-off location
 * payment_type	the payment method - credit card or cash
 * fare_amount	fare amount in dollars
 * surcharge	surcharge in dollars
 * mta_tax	tax in dollars
 * tip_amount	tip in dollars
 * tolls_amount	bridge and tunnel tolls in dollars
 * total_amount	total paid amount in dollars

Following are the two  lines from the data file:

```
   07290D3599E7A0D62097A346EFCC1FB5,E7750A37CAB07D0DFF0AF7E3573AC141,2013-01-01 00:00:00,2013-01-01 00:02:00,120,0.44,-73.956528,40.716976,-73.962440,40.715008,CSH,3.50,0.50,0.50,0.00,0.00,4.50
   22D70BF00EEB0ADC83BA8177BB861991,3FF2709163DE7036FCAA4E5A3324E4BF,2013-01-01 00:02:00,2013-01-01 00:02:00,0,0.00,0.000000,0.000000,0.000000,0.000000,CSH,27.00,0.00,0.50,0.00,0.00,27.50
```



### Query 1: Frequent Routes

Find the top 10 most frequent routes during the last 30 minutes. A route is represented by a starting grid cell and an ending grid cell. All routes completed within the last 30 minutes are considered for the query. 

### Query 2: Profitable Areas

 Identify areas that are currently most profitable for taxi drivers. The profitability of an area is determined by dividing the area profit by the number of empty taxis in that area within the last 15 minutes. The profit that originates from an area is computed by calculating the median fare + tip for trips that started in the area and ended within the last 15 minutes. The number of empty taxis in an area is the sum of taxis that had a drop-off location in that area less than 30 minutes ago and had no following pickup yet.

