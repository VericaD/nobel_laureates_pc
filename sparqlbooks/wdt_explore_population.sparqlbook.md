SPARQL Query for Fields of Work of Nobel Prize Winners in Physics
```sparql
PREFIX wd: <http://www.wikidata.org/entity/>  
PREFIX wdt: <http://www.wikidata.org/prop/direct/>  
PREFIX wikibase: <http://wikiba.se/ontology#>  

SELECT ?field ?fieldLabel (COUNT(*) AS ?n)  
WHERE {  
  ?laureate wdt:P166 wd:Q38104;  # Has received Nobel Prize in Physics  
            wdt:P101 ?field.     # Field of work  
  SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }  
}  
GROUP BY ?field ?fieldLabel  
ORDER BY DESC(?n)  
LIMIT 20  


````
