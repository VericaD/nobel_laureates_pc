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

#Top 5 field of work
# physics - 148 occurences
# theoretical physics - 30 occurences
# astrophysics - 15 occurences
# particle physics - 15 occurences
# nuclear physics - 12 occurences


````
Show and count how many properties are available for the Nobel prize winners population
```sparql
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

SELECT ?p ?propLabel ?eff
WHERE {
  {
    SELECT ?p (COUNT(*) AS ?eff)
    WHERE {
      ?item wdt:P166 wd:Q38104;  # Nobel Prize in Physics (award received)
            wdt:P31 wd:Q5;       # Must be a human
            #wdt:P569 ?birthDate; # Birth date
            ?p ?o.               # Any property associated with the person

      # Extract the birth year
      #BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
      
    }
    GROUP BY ?p 
  }

  # Get property labels
  ?prop wikibase:directClaim ?p .
  SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }  
}  
ORDER BY DESC(?eff)

# award received - 2627
# member of - 1315
# doctoral student - 1049
# occupation - 783
# employer - 711
# educated at - 641
# described by source - 572
# field of work - 454
# ...
````


