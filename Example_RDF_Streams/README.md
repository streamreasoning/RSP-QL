* Examples in the trig directory are the most current. The json-ld examples are legacy.
* Examples starting with "BGN" are RDF Stream examples that have bnode graph names.
* Examples starting with "RGN" are RDF Stream examples that have IRI graph names.
* Examples entitled "..._Location_TempC_Minute_(some city name).*" are RDF stream examples with 
data about temperature observations at a single location, given by the city name.
* Examples with "Merged" in the name are the result of the merger of other RDF streams, where the 
merging operation requires renaming of blank nodes when necessary to keep them distinct. 
E.g., "RGN_Location_TempC_Minute_Merged.*" is the result of merging "RGN_Location_TempC_Minute_Berlin.*",  "RGN_Location_TempC_Minute_Madrid.*",
and  "RGN_Location_TempC_Minute_Paris.*"
* Examples with "Union" in the name are the result of the union of other RDF streams, where the streams are allowed
to share blank nodes (i.e. they are on the same "surface").
* Examples with suffix "_b", "_c", ... are alternate serializations of the same RDF stream as the example without the suffix.
* Examples with "rank" in the name are not JSON-LD, but are an extended description that includes an integer "rank". This value corresponds to the number of timestamped graphs in the stream that are less than or equal to the given timestamped graph (including itself).
* Examples starting with HRA are a mockup of heart rate sensor readings in a 
  Discrete Timeseries with blank node named graphs.
* Examples named HRA_full have timestamps corresponding to the generated time, and the observation is contained in the named graph.
* Examples named HRA_empty have timestamps corresponding to the effective time, and the observation is contained in the default graph.
