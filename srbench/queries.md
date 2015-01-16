Srbench queries
===============

##Q1. Get the rainfall observed once in an hour.


  PREFIX om-owl: <http://knoesis.wright.edu/ssw/ont/sensor-observation.owl#>
  PREFIX weather: <http://knoesis.wright.edu/ssw/ont/weather.owl#>
  PREFIX srbench: <http://www.cwi.nl/SRBench/>
  SELECT ISTREAM ?sensor ?value ?uom
  FROM NAMED WINDOW ON STREAM srbench:observations [RANGE PT1H] AS :win
  WHERE {
    WINDOW :win { 
    ?observation om-owl:procedure ?sensor ;
                 a weather:RainfallObservation ;
                 om-owl:result ?result .
    ?result om-owl:floatValue ?value ;
            om-owl:uom ?uom .
  }}

##Q4. Get the average wind speed at the stations where the air temperature is >32 degrees in the last hour, every 10 minutes.

PREFIX om-owl: <http://knoesis.wright.edu/ssw/ont/sensor-observation.owl#>
PREFIX weather: <http://knoesis.wright.edu/ssw/ont/weather.owl#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT ISTREAM ?sensor (AVG(?windSpeed) AS ?averageWindSpeed)
               (AVG(?temperature) AS ?averageTemperature)
FROM NAMED WINDOW ON STREAM srbench:observations [RANGE PT3H SLIDE PT10M] AS :win
WHERE { 
  WINDOW :win {    
  ?temperatureObservation om-owl:procedure ?sensor ;
                          a weather:TemperatureObservation ;
                          om-owl:result ?temperatureResult .
  ?temperatureResult om-owl:floatValue ?temperature ;
                     om-owl:uom ?uom .
  FILTER(?temperature > "32"^^xsd:float )
  ?windSpeedObservation om-owl:procedure ?sensor ;
                        a weather:WindSpeedObservation ;
                        om-owl:result [ om-owl:floatValue ?windSpeed ]  .
}}
GROUP BY ?sensor

##Q5. Detect if a station is observing a blizzard.

PREFIX om-owl: <http://knoesis.wright.edu/ssw/ont/sensor-observation.owl#>
PREFIX weather: <http://knoesis.wright.edu/ssw/ont/weather.owl#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

CONSTRUCT ISTREAM{ ?sensor om-owl:generatedObservation [a weather:Blizzard] }
FROM NAMED WINDOW ON STREAM srbench:observations [RANGE PT3H SLIDE PT10M] AS :win
WHERE {
  WINDOW :win { 
    SELECT ?sensor
    WHERE {
      ?sensor om-owl:generatedObservation [a weather:SnowfallObservation] ;
              om-owl:generatedObservation ?o1 ;
              om-owl:generatedObservation ?o2 .
      ?o1 a weather:TemperatureObservation ;
          om-owl:observedProperty weather:_AirTemperature ;
          om-owl:result [om-owl:floatValue ?temperature] .
      ?o2 a weather:WindObservation ;
          om-owl:observedProperty weather:_WindSpeed ; 
          om-owl:result [om-owl:floatValue ?windSpeed] .
    }
    GROUP BY ?sensor
    HAVING ( AVG(?temperature) < "32"^^xsd:float  &&  # fahrenheit
             MIN(?windSpeed) > "40.0"^^xsd:float ) #milesPerHour
  }
}

##Q10. Get the locations where a heavy snowfall has been observed in the last day.

PREFIX om-owl: <http://knoesis.wright.edu/ssw/ont/sensor-observation.owl#>
PREFIX weather: <http://knoesis.wright.edu/ssw/ont/weather.owl#>
PREFIX wgs84_pos: <http://www.w3.org/2003/01/geo/wgs84_pos>

SELECT ISTREAM ?lat ?long ?alt
FROM NAMED WINDOW ON STREAM srbench:observations [RANGE PT1H] AS :win
FROM <http://www.cwi.nl/SRBench/sensors>
WHERE {
  WINDOW :w {
    ?sensor om-owl:generatedObservation [a weather:SnowfallObservation] .
  }
  ?sensor om-owl:processLocation ?sensorLocation .
  ?sensorLocation wgs84_pos:alt ?alt ;
                  wgs84_pos:lat ?lat ;
                  wgs84_pos:long ?long .
}
