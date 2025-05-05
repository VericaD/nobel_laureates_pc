# Import citizenships
In this notebook we add to the imported Wikidata population properties regarding citizenship in countries
```sparql
### Number of persons in our population
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        {?s a wd:Q5.}
}

````
In total 770
````sparql
### Some examples of persons
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT ?s ?label ?birthYear
WHERE {
    GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        {?s a wd:Q5;
            rdfs:label ?label;
            wdt:P569 ?birthYear}
}
ORDER BY ?s
LIMIT 3

````
````sparql
### Prepare and inspect the data to be imported


PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

CONSTRUCT {?item wdt:P27 ?citizenship.
            ?citizenship rdfs:label ?citizenshipLabel}
WHERE
    {
        GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>

        ## Find the persons in the imported graph
        {SELECT ?item
        WHERE 
                {?item a wd:Q5.}
        ORDER BY ?item      
        OFFSET 0
        #OFFSET 10000
        LIMIT 10

        }
        ## 
        SERVICE <https://query.wikidata.org/sparql>
            {
                ?item wdt:P27 ?citizenship.
                BIND (?citizenshipLabel as ?citizenshipLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
            }
                
        }

````
````sparql
### To be sure, the insert query has to be carried out directly on the Allegrograph server
# but it also could work if executed in this notebook
## Also, you have to carry it out in three steps. The accepted limit by Wikidata 
## of instances in a variable ('item' in our case) appears to be 10000.
## You therefore have to have three steps for a population of around 23000 persons  

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>


WITH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
INSERT {?item wdt:P27 ?citizenship.}
WHERE
    {
        ## Find the persons in the imported graph
        {SELECT ?item
        WHERE 
                {?item a wd:Q5.}
        ORDER BY ?item      
        #OFFSET 8000
        #OFFSET 16000
        #OFFSET 24000
        #OFFSET 32000
        LIMIT 10000

        }
        ## 
        SERVICE <https://query.wikidata.org/sparql>
            {
                ?item wdt:P27 ?citizenship.
            }
                
        }
````
````sparql
### Insert the label of the property
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>


INSERT DATA {
  GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
  {wdt:P27 rdfs:label 'country of citizenship'.}
}
````
````sparql
### Get the number of created 'citizenships'
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>


    SELECT (COUNT(*) as ?n) 
    WHERE {
        GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
            {
                ?s wdt:P27 ?o.
            }
            }
    
````
````sparql
### Persons without a country of citizenship
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT (COUNT(*) as ?n)
WHERE 
{GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        
    {?item a wd:Q5;
        rdfs:label ?label.
    MINUS {
            ?item wdt:P27 ?country   .
        }     
    }
}
# There is not a person without a citizenship
````
````sparql
### Persons without a country of citizenship
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT ?item ?label
WHERE 
{GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        
    {?item a wd:Q5;
        rdfs:label ?label.
    MINUS {
            ?item wdt:P27 ?country   .
        }     
    }
}
OFFSET 10000
LIMIT 10

# There is no such person
````
Query modification
````sparql
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT ?citizenship (COUNT(*) as ?n) WHERE {
  GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md> {
    ?item wdt:P27 ?citizenship .
  }
}
GROUP BY ?citizenship
ORDER BY DESC(?n)
LIMIT 5
````
### Inspect the available information
````sparql
### Basic query about persons' properties

PREFIX franzOption_defaultDatasetBehavior: <franz:rdf>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?p ?label (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        {?s a wd:Q5;
            ?p ?o.
        OPTIONAL {?p rdfs:label ?label}    
          }
}
GROUP BY ?p ?label
ORDER BY DESC(?n)

````
### Enrich the information available in your graph about countries
````sparql
### Get the labels of the countries 
# Prepare the insert

PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

CONSTRUCT  {
    ?country rdfs:label ?countryLabel.
}
#SELECT DISTINCT ?country ?countryLabel
WHERE {

    {
    SELECT DISTINCT ?country
    WHERE {
        GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
            {
                ?s wdt:P27 ?country.
            }
            }
    LIMIT 5
    }

    SERVICE <https://query.wikidata.org/sparql>
                {
                BIND (?country as ?country)
                BIND (?countryLabel as ?countryLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
                }



}
````
````sparql
### Execute the INSERT, from the sparqlbook or on Allegrograph

PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

WITH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md> 
INSERT  {
    ?citizenship rdfs:label ?citizenshipLabel.
}
WHERE {

    {
    SELECT DISTINCT ?citizenship
    WHERE {
            {
                ?s wdt:P27 ?citizenship.
            }
          }
    }

    SERVICE <https://query.wikidata.org/sparql>
                {
                BIND (?citizenship as ?citizenship)
                BIND (?citizenshipLabel as ?citizenshipLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
                }



}
````
````sparql
###  Inspect the citizenships
# number of persons having this citizenship

PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>


SELECT ?citizenship ?citizenshipLabel (COUNT(*) as ?n)
WHERE {
GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
{
   ?s wdt:P27 ?citizenship.
   ?citizenship rdfs:label ?citizenshipLabel.
}

}
GROUP BY ?citizenship ?citizenshipLabel
ORDER BY DESC(?n)
limit 20

# Q30 "Unated States" 333
# Q145 "Unated Kingdom" 108
# Q183 "Germany" 72
#...
````
````sparql
````
````sparql
````
````sparql
````
````sparql
````
````sparql
````
````sparql
````
