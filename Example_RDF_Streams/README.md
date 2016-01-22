* Examples starting with "BGN" are RDF Stream examples that have bnode graph names.
* Examples starting with "RGN" are RDF Stream examples that have IRI graph names.
* Examples entitled "..._Location_TempC_Minute_(some city name).json" are RDF stream examples with 
data about temperature observations at a single location, given by the city name.
* Examples with "Merged" in the name are the result of the merger of other RDF streams, where the 
merging operation requires renaming of blank nodes when necessary to keep them distinct. 
E.g., "RGN_Location_TempC_Minute_Merged.json" is the result of merging "RGN_Location_TempC_Minute_Berlin.json",  "RGN_Location_TempC_Minute_Madrid.json",
and  "RGN_Location_TempC_Minute_Paris.json"
* Examples with "Union" in the name are the result of the union of other RDF streams, where the streams are allowed
to share blank nodes (i.e. they are on the same "surface").
* Examples with suffix "_b", "_c", ... are alternate serializations of the same RDF stream as the example without the suffix.
* Examples containing "rank" are not JSON-LD, but are an extended description that includes an integer "rank". This value corresponds to the number of timestamped graphs in the stream that are less than or equal to the given timestamped graph (including itself).
