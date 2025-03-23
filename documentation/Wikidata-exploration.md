SPARQL queries made in Wikidata
The number of Nobel Prize in Physics winners (in total 226)
```sparql
SELECT (COUNT(*) AS ?eff)
WHERE {
  ?item wdt:P31 wd:Q5;  # Any instance of a human (Q5 is for humans).
  wdt:P166 wd:Q38104 .  # Nobel Prize in Physics.
}


