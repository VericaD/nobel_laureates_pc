# Wikidata exploration
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
      { ?item wdt:P166 wd:Q38104 }  # Nobel Prize in Physics
      UNION
      { ?item wdt:P166 wd:Q44585 }  # Nobel Prize in Chemistry
      UNION
      { ?item wdt:P166 wd:Q80061 }  # Nobel Prize in Physiology or Medicine
      UNION
      { ?item wdt:P166 wd:Q37922 }  # Nobel Prize in Literature

      ?item wdt:P31 wd:Q5; # Must be a human
            ?p ?o.         # Any property associated with them
    }
    GROUP BY ?p 
  }

  # Get property labels
  ?prop wikibase:directClaim ?p .
  SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }  
}  
ORDER BY DESC(?eff)


# award received - 2627 - occurrences
# member of - 1315 - occurrences
# doctoral student - 1049 - occurrences
# occupation - 783 - occurrences
# employer - 711 - occurrences
# educated at - 641 - occurrences
# described by source - 572 - occurrences
# field of work - 454 - occurrences
# ...
````

SPARQL Query for Fields of Work of Nobel Prize Winners
```sparql
PREFIX wd: <http://www.wikidata.org/entity/>  
PREFIX wdt: <http://www.wikidata.org/prop/direct/>  
PREFIX wikibase: <http://wikiba.se/ontology#>  

SELECT ?field ?fieldLabel (COUNT(*) AS ?n)  
WHERE {  
  {?laureate wdt:P166 wd:Q38104;  # Nobel Prize in Physics  
              wdt:P101 ?field.    # Field of work
  }
  UNION
  {?laureate wdt:P166 wd:Q44585;  # Nobel Prize in Chemistry  
              wdt:P101 ?field.    # Field of work
  }
  UNION
  {?laureate wdt:P166 wd:Q80061;  # Nobel Prize in Physiology or Medicine  
              wdt:P101 ?field.    # Field of work
  }
  UNION
  {?laureate wdt:P166 wd:Q37922;  # Nobel Prize in Literature  
              wdt:P101 ?field.    # Field of work
  }

  SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }  
}  
GROUP BY ?field ?fieldLabel  
ORDER BY DESC(?n)  
LIMIT 20


#Top 5 field of work
# physics - 148 occurrences
# theoretical physics - 30 occurrences
# astrophysics - 15 occurrences
# particle physics - 15 occurrences
# nuclear physics - 12 occurrences


````

Retrieves a list of Nobel Prize laureates and their birth in increasing order
```sparql
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wikibase: <http://wikiba.se/ontology#>

SELECT ?laureate ?laureateLabel ?birthDate
WHERE {
  ?laureate wdt:P166 ?prize;
            wdt:P569 ?birthDate.
  FILTER (?prize IN (wd:Q38104, wd:Q44585, wd:Q80061, wd:Q37922))  # Filtering Nobel Prize categories
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
ORDER BY ?birthDate


```
The number of Nobel Prize winners (in total 771)
```sparql
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wd: <http://www.wikidata.org/entity/>

SELECT (COUNT(*) AS ?eff)
WHERE {
  ?item wdt:P31 wd:Q5;  # Any instance of a human (Q5 is for humans).
        wdt:P166 ?prize.  
  FILTER (?prize IN (wd:Q38104, wd:Q44585, wd:Q80061, wd:Q37922))  # Nobel Prizes in Physics, Chemistry, Medicine, and Literature
}

````
Nobel Prize Winners in Physics by Country
````sparql
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

SELECT ?country ?countryLabel (COUNT(DISTINCT ?winner) AS ?numWinners)
WHERE {
  ?winner wdt:P166 wd:Q38104;  # Nobel Prize in Physics
          wdt:P27 ?country.    # Country of citizenship

  SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }  # Get country names in English
}
GROUP BY ?country ?countryLabel
ORDER BY DESC(?numWinners)  # Sort by highest number of winners
````


