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


# wdt:P166	award received	         8568
# wdt:P463	member of	               3814
# wdt:P106	occupation	             3438
# wdt:P1343	described by source	     2090
# wdt:P69	educated at	               2001
# wdt:P108	employer	             1874
# wdt:P185	doctoral student	     1700
# wdt:P101	field of work	         1409
# wdt:P27	country of citizenship	   1098
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
# wd:Q413	physics	               166
# wd:Q7094	biochemistry	       112
# wd:Q2329	chemistry	        85
# wd:Q521	physiology	        32
# wd:Q18362	theoretical physics	31


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


