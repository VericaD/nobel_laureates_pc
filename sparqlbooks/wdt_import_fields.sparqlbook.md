# Import fields of activity of persons

````sparql
### Number of persons 
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        {?s a wd:Q5.}
}

````
````sparql
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
## Fields 
````sparql
## Explore all fields

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

SELECT ?item ?birthYear ?field ?fieldLabel 
WHERE
    {
        ## Find the persons in the imported graph
        {SELECT ?item ?birthYear
        WHERE 
                {?item a wd:Q5;
                    wdt:P569 ?birthYear}
        ORDER BY ?item      
        #OFFSET 10000
        #OFFSET 20000
        LIMIT 100

        }
        ## 
        SERVICE <https://query.wikidata.org/sparql>
            {
                ?item wdt:P101 ?field.
                BIND (?fieldLabel as ?fieldLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
            }
                
        }
ORDER BY ?item ?field
limit 20

````
````sparql
## Explore fields

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

SELECT (COUNT(*) as ?n)
WHERE
    {
        ## Find the persons in the imported graph
        {SELECT ?item ?birthYear
        WHERE 
                {?item a wd:Q5}
        ORDER BY ?item      
        #OFFSET 10000
        #OFFSET 20000
       LIMIT 10000

        }
        ## 
        SERVICE <https://query.wikidata.org/sparql>
            {
                ?item wdt:P101 ?field .
            }
                
        }

````
````sparql
## Group and count

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

SELECT ?field ?fieldLabel (COUNT(*) as ?n)
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
        #OFFSET 20000
       LIMIT 10000

        }
        ## 
        SERVICE <https://query.wikidata.org/sparql>
            {
                ?item wdt:P101 ?field.
                BIND (?fieldLabel as ?fieldLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
            }
                
        }
GROUP BY ?field ?fieldLabel
ORDER BY DESC(?n)
LIMIT 20

````
````sparql
## Group and count

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

SELECT ?field ?fieldLabel ?fieldClassLabel (COUNT(*) as ?n)
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
        #OFFSET 20000
       LIMIT 10000

        }
        ## 
        SERVICE <https://query.wikidata.org/sparql>
            {
                # field
                ?item wdt:P101 ?field.
                # instance of
                ?field wdt:P31 ?fieldClass.
                BIND (?fieldLabel as ?fieldLabel)
                BIND (?fieldClassLabel as ?fieldClassLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
            }
                
        }
GROUP BY ?field ?fieldLabel ?fieldClassLabel
ORDER BY DESC(?n)
LIMIT 20

````
### Import data
````sparql
### Prepare the data to be imported
# With LIMIT clause 
## Apparently labels are not repeated if already available
# We therefore car integrate them directly in the INSERT below

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

CONSTRUCT {?item wdt:P101 ?field.
            ?field rdfs:label ?fieldLabel}
WHERE
    {
        ## Find the persons in the imported graph
        {SELECT ?item
        WHERE 
                {?item a wd:Q5.}
        ORDER BY ?item      
        OFFSET 0
        #OFFSET 10000
        #OFFSET 20000
        LIMIT 7

        }
        ## 
        SERVICE <https://query.wikidata.org/sparql>
            {
                ?item wdt:P101 ?field.
                BIND (?fieldLabel as ?fieldLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
            }
                
        }
````
````sparql
### This insert query has to be carried out directly on the Allegrograph server

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

WITH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
INSERT {?item wdt:P101 ?field.
         ?field rdfs:label ?fieldLabel}
WHERE
    {
        ## Find the persons in the imported graph
        {SELECT ?item
        WHERE 
                {?item a wd:Q5.}
        ORDER BY ?item      
        #OFFSET 10000
        #OFFSET 20000
        #OFFSET 30000
        LIMIT 10000

        }
        ## 
        SERVICE <https://query.wikidata.org/sparql>
            {
                ?item wdt:P101 ?field.
                BIND (?fieldLabel as ?fieldLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
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
  {wdt:P101 rdfs:label 'field'.}
}
````
### Add the field class
````sparql
###  Inspect the fields:
# number of different fields 

PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT (COUNT(*) as ?n)
WHERE
   {
   SELECT DISTINCT ?field
   WHERE {
      GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
         {
            ?s wdt:P101 ?field.
         }
      }
   }

# number of different fields : 381
````
````sparql
### Insert the class 'field of work' for all the fields
# Please note that strictly speaking Wikidata has no ontology,
# therefore no classes. We add this for our convenience
# Also, Wikidata 'fields' are not explicitly associated 
# with the Q627436 'Field of work' concept. We retain this concept
# as more global than the parly overlapping Q1047113 'field of study'

PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

WITH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
INSERT {
   ?field rdf:type wd:Q627436.
}
WHERE
   {
   SELECT DISTINCT ?field
   WHERE {
         {
            ?s wdt:P101 ?field.
         }
      }
   }
````
````sparql
### Add label for concept field

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>

INSERT DATA {
GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
    {    wd:Q627436 rdfs:label "Field of Work".
    }    
}


````
### Inspect the available information
````sparql
### Basic query about persons' field

PREFIX franzOption_defaultDatasetBehavior: <franz:rdf>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?field ?fieldLabel (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        {?item wdt:P101 ?field.
        OPTIONAL {?field rdfs:label ?fieldLabel}    
          }
}
GROUP BY ?field ?fieldLabel 
ORDER BY DESC(?n)
LIMIT 10
````
# The following part is to be repeated
## Get the fields' parent fields ('classes')
Given the dispersion of fields, we try to obtain parent terms of fields to see if some groupings are possible.
````sparql
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

SELECT (COUNT(*) as ?n)
WHERE {
        GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        {?item a wd:Q12737077.}
        }
         
````
````sparql
## 

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

SELECT ?field ?fieldLabel (COUNT(*) as ?n)
WHERE
    {
        ## Find the fields in the imported graph
        {SELECT DISTINCT ?item
        WHERE 
            {?item a wd:Q12737077.}
        ORDER BY ?item      
        LIMIT 10000

        }
        ## 
        SERVICE <https://query.wikidata.org/sparql>
            {
                # subclass of
                ?item wdt:P279 ?field.
                BIND (?fieldLabel as ?fieldLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
            }
                
        }
GROUP BY ?field ?fieldLabel
ORDER BY DESC(?n)
LIMIT 20

````
````sparql
## Prepare import

PREFIX franzOption_defaultDatasetBehavior: <franz:rdf>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

CONSTRUCT {
    ?item  wdt:P279 ?field.
    ?field rdfs:label ?fieldLabel.
    # rdf:type field
    ?field a wd:Q12737077.
    }
WHERE
    {
        GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        ## Find the persons in the imported graph
        {SELECT ?item
        WHERE 
                {?item a wd:Q12737077.}
        ORDER BY ?item      
        LIMIT 10000

        }
        ## 
        SERVICE <https://query.wikidata.org/sparql>
            {
                # subclass of
                ?item wdt:P279 ?field.
                BIND (?fieldLabel as ?fieldLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
            }
                
        }
ORDER BY DESC(?item)
LIMIT 5

````
### Insert parent fields
````sparql
### Insert the label of the property

## This is needed just once

PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>


INSERT DATA {
  GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
  {wdt:P279 rdfs:label 'subclass of'.}
}
````
````sparql
### INSERT

## IMPORTANT : execute the classification levels' queries below 
# after each insert and observe what happens

PREFIX franzOption_defaultDatasetBehavior: <franz:rdf>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

WITH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
INSERT {
    ?item  wdt:P279 ?field.
    ?field rdfs:label ?fieldLabel.
    # rdf:type field
    ?field a wd:Q12737077.
    }
WHERE
    {
        ## Find the persons in the imported graph
        {SELECT ?item
        WHERE 
                {?item a wd:Q12737077.}
        ORDER BY ?item      
        LIMIT 10000

        }
        ## 
        SERVICE <https://query.wikidata.org/sparql>
            {
                # subclass of
                ?item wdt:P279 ?field.
                BIND (?fieldLabel as ?fieldLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
            }
                
        }



````
### Inspect imported data
````sparql
### Propriétés des fields: outgoing

PREFIX franzOption_defaultDatasetBehavior: <franz:rdf>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?p ?label (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        {?s a wd:Q12737077;
            ?p ?o.
        OPTIONAL {?p rdfs:label ?label}    
          }
}
GROUP BY ?p ?label
ORDER BY DESC(?n)
````
````sparql
### field classifications

PREFIX franzOption_defaultDatasetBehavior: <franz:rdf>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?birthYear ?o ?oLabel ?p ?o1 ?o1Label
WHERE {
    GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        {?item a wd:Q5;
            wdt:P569 ?birthYear;
            wdt:P106 ?o.
        ?o a wd:Q12737077;
            wdt:P279 ?o1.
        OPTIONAL {?o rdfs:label ?oLabel}    
        OPTIONAL {?o1 rdfs:label ?o1Label}    
          }
}
ORDER BY ?s
LIMIT 10
````
### First classification level
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
````sparql
````
````sparql
````

