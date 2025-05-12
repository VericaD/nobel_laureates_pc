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
### First classification level

# filled after first insert

PREFIX franzOption_defaultDatasetBehavior: <franz:rdf>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?parentfield ?parentfieldlabel (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        {?s a wd:Q5.
        
        # ?s wdt:P106 ?field.
        # ?field wdt:P279 ?parentfield
        ## property path:
        ?s  wdt:P106/wdt:P279  ?parentfield

        OPTIONAL {?parentfield rdfs:label ?parentfieldlabel}    
          }
}
GROUP BY ?parentfield ?parentfieldlabel 
ORDER BY DESC(?n)
LIMIT 20
````
### Second classification level
````sparql
### Second classification level

# no result after first insert
# result after first insert

PREFIX franzOption_defaultDatasetBehavior: <franz:rdf>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?parentfield ?parentfieldlabel (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        {?s a wd:Q5.
        
        ## property path:
        ?s  wdt:P106/wdt:P279/wdt:P279  ?parentfield

        OPTIONAL {?parentfield rdfs:label ?parentfieldlabel}    
          }
}
GROUP BY ?parentfield ?parentfieldlabel 
ORDER BY DESC(?n)
LIMIT 20
````
Third classification level
````sparql
### Third classification level

# no result after first insert
# result after first insert ! 



PREFIX franzOption_defaultDatasetBehavior: <franz:rdf>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?parentfield ?parentfieldlabel (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        {?s a wd:Q5.
        
        ## property path:
        ?s  wdt:P106/wdt:P279/wdt:P279/wdt:P279  ?parentfield

        OPTIONAL {?parentfield rdfs:label ?parentfieldlabel}    
          }
}
GROUP BY ?parentfield ?parentfieldlabel 
ORDER BY DESC(?n)
LIMIT 20
````
````sparql
### Third classification level

# explicit query path to check
# that there are no recursions



PREFIX franzOption_defaultDatasetBehavior: <franz:rdf>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?parentfield ?parentfieldlabel (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        {?s a wd:Q5.
        
        ## property path:
        ?s  wdt:P106 ?c1.
        ?c1 wdt:P279 ?c2.
        ?c2 wdt:P279 ?c3.
        ?c3 wdt:P279  ?parentfield
        ## Avoid recursivity
        FILTER ( ?c2 != ?c1)
        FILTER ( ?c3 not in (?c1, ?c2) )
        FILTER (  ?parentfield not in (?c1, ?c2, ?c3) )

        OPTIONAL {?parentfield rdfs:label ?parentfieldlabel}    
          }
}
GROUP BY ?parentfield ?parentfieldlabel 
ORDER BY DESC(?n)
LIMIT 20
````
Fourth classification level
````sparql
### Fourth classification level

# no result after first insert
# result after first insert ! 



PREFIX franzOption_defaultDatasetBehavior: <franz:rdf>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?parentfield ?parentfieldlabel (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        {?s a wd:Q5.
        
        ## property path:
        ?s  wdt:P106 ?c1.
        ?c1 wdt:P279 ?c2.
        ?c2 wdt:P279 ?c3.
        ?c3 wdt:P279 ?c4.
        ?c4 wdt:P279  ?parentfield
        FILTER ( ?c2 != ?c1)
        FILTER ( ?c3 not in (?c1, ?c2) )
        FILTER ( ?c4 not in (?c1, ?c2, ?c3) )
        FILTER (  ?parentfield not in (?c1, ?c2, ?c3, ?c4) )

        OPTIONAL {?parentfield rdfs:label ?parentfieldlabel}    
          }
}
GROUP BY ?parentfield ?parentfieldlabel 
ORDER BY DESC(?n)
LIMIT 20
````
### Test: get rid of the classifications and restart
````sparql
### First classification level

PREFIX franzOption_defaultDatasetBehavior: <franz:rdf>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?s ?o ?oClass
WHERE {
    GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        {
            ## retrieves all the classifications:
            ?s wdt:P279 ?o.
            ?o a ?oClass
          }
}
LIMIT 20
````
I didn't do the DELET
````sparql
````
### Inspect the instance of metaclasses to fields
````sparql
## 

PREFIX franzOption_defaultDatasetBehavior: <franz:rdf>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

SELECT ?field ?fieldLabel ?metaclass ?metaclassLabel ?n
WHERE
    {
        GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        ## Find the persons in the imported graph
        {SELECT ?field ?fieldLabel (COUNT(*) as ?n)
        WHERE 
                {?item a wd:Q5.
                ?item wdt:P106 ?field.
                ?field rdfs:label ?fieldLabel}
        GROUP BY ?field ?fieldLabel      

        }
        ## 
        SERVICE <https://query.wikidata.org/sparql>
            {
                # instance of
                ?field wdt:P31 ?metaclass.
                BIND (?metaclassLabel as ?metaclassLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
            }
                
        }
ORDER BY DESC(?n)
LIMIT 30

````
## Inspect parent fields 
The aim is to find a way to distinguish between fields
````sparql
### Find fields and parent fields of parent fields

PREFIX franzOption_defaultDatasetBehavior: <franz:rdf>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

SELECT ?field ?fieldLabel
          ?parentfield ?parentfieldlabel 
          ?fieldField ?fieldFieldLabel
          ?knowledgeClassification ?knowledgeClassificationLabel
          ?parentKnowledgeClassification ?parentKnowledgeClassificationLabel
          (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        {SELECT ?field ?fieldLabel
          ?parentfield ?parentfieldlabel 
          ?parentKnowledgeClassification ?parentKnowledgeClassificationLabel
        
        WHERE
        {?s a wd:Q5.
        
        ?s  wdt:P106 ?field.
        ?field  wdt:P279  ?parentfield.
        ?field rdfs:label ?fieldLabel.
        ?parentfield rdfs:label ?parentfieldlabel    
        }
        LIMIT 3000
        }

        SERVICE <https://query.wikidata.org/sparql>
          {
              # field of this field
              ?parentfield wdt:P425 ?fieldField.
              # instance of
              ?fieldField wdt:P31 ?knowledgeClassification.
              # subclass of
              ?knowledgeClassification wdt:P279 ?parentKnowledgeClassification.


              BIND (?fieldFieldLabel as ?fieldFieldLabel)
              BIND (?knowledgeClassificationLabel as ?knowledgeClassificationLabel)
              BIND (?parentKnowledgeClassificationLabel as ?parentKnowledgeClassificationLabel)
              SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
          }

}
GROUP BY ?field ?fieldLabel
          ?parentfield ?parentfieldlabel 
          ?fieldField ?fieldFieldLabel
          ?knowledgeClassification ?knowledgeClassificationLabel
          ?parentKnowledgeClassification ?parentKnowledgeClassificationLabel
ORDER BY DESC(?n)
LIMIT 20
````
````sparql
### Simplified query

PREFIX franzOption_defaultDatasetBehavior: <franz:rdf>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

SELECT DISTINCT 
?field ?fieldLabel ?parentField ?parentFieldLabel
?n
          #?parentKnowledgeClassification ?parentKnowledgeClassificationLabel 
          # (SUM(?n) as ?sn)
WHERE {
    GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        {
          SELECT ?field ?fieldLabel
          ?parentfield (COUNT(*) as ?n)
        
        WHERE
        {
          ?s a wd:Q5;  
              wdt:P106 ?field.
          ?field rdfs:label ?fieldLabel.
          ?field  wdt:P279  ?parentfield.
          }  
        GROUP BY ?field ?fieldLabel
          ?parentfield
        }

        SERVICE <https://query.wikidata.org/sparql>
          {
              # field of parent field / instance of /  subclass of  
              ?parentfield wdt:P425 / wdt:P31 ?parentField.
              # ?parentField wdt:P279 ?parentKnowledgeClassification.


              #BIND (?field as ?fieldLabel)
              BIND (?parentFieldLabel as ?parentFieldLabel)
              # BIND (?parentKnowledgeClassificationLabel as ?parentKnowledgeClassificationLabel)
              SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
          }

}
# GROUP BY ?field ?fieldLabel ?parentField ?parentFieldLabel
          # ?parentKnowledgeClassification ?parentKnowledgeClassificationLabel 
ORDER BY DESC(?n)
LIMIT 30
````
````sparql
### Create pairs of fields 
# and add the birth year of the person

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

SELECT ?item ?birthYear ?field ?fieldLabel ?field_1 ?field_1Label
WHERE
    {
        GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>

        ## Find the persons in the imported graph
        {SELECT ?item ?birthYear
        WHERE 
                {?item a wd:Q5;
                    wdt:P569 ?birthYear}
        ORDER BY ?item      
        OFFSET 0
        #OFFSET 10000
        #OFFSET 20000
       LIMIT 5

        }
        ## 
        SERVICE <https://query.wikidata.org/sparql>
            {
                ?item wdt:P106 ?field.
                ?item wdt:P106 ?field_1.
                FILTER (str(?fieldLabel) < str(?field_1Label))
                BIND (?fieldLabel as ?fieldLabel)
                BIND (?field_1Label as ?field_1Label)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
            }
                
        }


````



