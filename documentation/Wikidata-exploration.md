## SPARQL queries made in Wikidata
Retrieves a list of Nobel Prize in Physics laureates and their birth in increasing order
```sparql
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wd: <http://www.wikidata.org/entity/>

SELECT ?laureate ?laureateLabel ?birthDate
WHERE {
  ?laureate wdt:P166 wd:Q38104;  
           wdt:P569 ?birthDate.  
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }  # Get labels in English
}
ORDER BY ?birthDate  # To order the results by birth date

```
The number of Nobel Prize in Physics winners (in total 226)
```sparql
SELECT (COUNT(*) AS ?eff)
WHERE {
  ?item wdt:P31 wd:Q5;  # Any instance of a human (Q5 is for humans).
  wdt:P166 wd:Q38104 .  # Nobel Prize in Physics.
}


