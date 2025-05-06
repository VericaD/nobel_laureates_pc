# Import occupations
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
## Occupations
````sparql
## Explore all the occupations

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

SELECT ?item ?birthYear ?occupation ?occupationLabel 
WHERE
    {
        ## Find the persons in the imported graph
        {SELECT ?item ?birthYear
        WHERE 
                {?item a wd:Q5;
                    wdt:P569 ?birthYear}
        ORDER BY ?item      
        OFFSET 0
        #OFFSET 10000
        #OFFSET 20000
       LIMIT 100

        }
        ## 
        SERVICE <https://query.wikidata.org/sparql>
            {
                ?item wdt:P106 ?occupation.
                BIND (?occupationLabel as ?occupationLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
            }
                
        }
ORDER BY ?item ?occupation
limit 20

````
````sparql
## Group and count

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

SELECT ?occupation ?occupationLabel (COUNT(*) as ?n)
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
                ?item wdt:P106 ?occupation.
                BIND (?occupationLabel as ?occupationLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
            }
                
        }
GROUP BY ?occupation ?occupationLabel
ORDER BY DESC(?n)
LIMIT 20

# University teacher: 475
# Physicist: 282
# Chemist: 265
# ...
````
## Pairs of occupations
````sparql
### Create pairs of occupations of the same person
# and add the birth year of the person

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

SELECT ?item ?birthYear ?occupation ?occupationLabel ?occupation_1 ?occupation_1Label
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
                ?item wdt:P106 ?occupation.
                ?item wdt:P106 ?occupation_1.
                FILTER (str(?occupationLabel) < str(?occupation_1Label))
                BIND (?occupationLabel as ?occupationLabel)
                BIND (?occupation_1Label as ?occupation_1Label)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
            }
                
        }


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

CONSTRUCT {?item wdt:P106 ?occupation.
            ?occupation rdfs:label ?occupationLabel}
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
        LIMIT 10

        }
        ## 
        SERVICE <https://query.wikidata.org/sparql>
            {
                ?item wdt:P106 ?occupation.
                BIND (?occupationLabel as ?occupationLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
            }
                
        }
````
````sparql
### This insert query has to be carried out directly on the Allegrograph server
## Also, you have to carry it out in three steps. The accepted limit by Wikidata 
## of instances in a variable ('item' in our case) appears to be 10000.
## You therefore have to have three steps for a population of around 23000 persons  

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

WITH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
INSERT {?item wdt:P106 ?occupation.
         ?occupation rdfs:label ?occupationLabel}
WHERE
    {
        ## Find the persons in the imported graph
        {SELECT ?item
        WHERE 
                {?item a wd:Q5.}
        ORDER BY ?item      
        #OFFSET 0
        #OFFSET 10000
        #OFFSET 20000
        OFFSET 30000
        LIMIT 10000

        }
        ## 
        SERVICE <https://query.wikidata.org/sparql>
            {
                ?item wdt:P106 ?occupation.
                BIND (?occupationLabel as ?occupationLabel)
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
  {wdt:P106 rdfs:label 'occupation'.}
}
````
### Add the Occupation class
````sparql
###  Inspect the occupations:
# number of different occupations

PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT (COUNT(*) as ?n)
WHERE
   {
   SELECT DISTINCT ?occupation
   WHERE {
      GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
         {
            ?s wdt:P106 ?occupation.
         }
      }
   }

# The number of different occupations is 261
````
````sparql
### Insert the class 'occupation' for all the occupations
# Please note that strictly speaking Wikidata has no ontology,
# therefore no classes. We add this for our convenience

PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

WITH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
INSERT {
   ?occupation rdf:type wd:Q12737077.
}
WHERE
   {
   SELECT DISTINCT ?occupation
   WHERE {
         {
            ?s wdt:P106 ?occupation.
         }
      }
   }
````
````sparql
### Add label for the Occupation concept

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>

INSERT DATA {
GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
    {    wd:Q12737077 rdfs:label "Occupation".
    }    
}


````
### Inspect the available information
````sparql
### Basic query about persons' occupation

PREFIX franzOption_defaultDatasetBehavior: <franz:rdf>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?occupation ?occupationLabel (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        {?item wdt:P106 ?occupation.
        OPTIONAL {?occupation rdfs:label ?occupationLabel}    
          }
}
GROUP BY ?occupation ?occupationLabel 
ORDER BY DESC(?n)
LIMIT 10
````
## Get the occupations' parent occupations ('classes')
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

SELECT ?occupation ?occupationLabel (COUNT(*) as ?n)
WHERE
    {
        ## Find the occupations in the imported graph
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
                ?item wdt:P279 ?occupation.
                BIND (?occupationLabel as ?occupationLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
            }
                
        }
GROUP BY ?occupation ?occupationLabel
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
    ?item  wdt:P279 ?occupation.
    ?occupation rdfs:label ?occupationLabel.
    # rdf:type Occupation
    ?occupation a wd:Q12737077.
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
                ?item wdt:P279 ?occupation.
                BIND (?occupationLabel as ?occupationLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
            }
                
        }
ORDER BY DESC(?item)
LIMIT 5

````
### Insert parent occupations
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
    ?item  wdt:P279 ?occupation.
    ?occupation rdfs:label ?occupationLabel.
    # rdf:type Occupation
    ?occupation a wd:Q12737077.
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
                ?item wdt:P279 ?occupation.
                BIND (?occupationLabel as ?occupationLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
            }
                
        }



````
### Inspect imported data
````sparql
### Occupation Properties: outgoing

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
### Occupation classifications

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

SELECT ?parentOccupation ?parentOccupationlabel (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        {?s a wd:Q5.
        
        # ?s wdt:P106 ?occupation.
        # ?occupation wdt:P279 ?parentOccupation
        ## property path:
        ?s  wdt:P106/wdt:P279  ?parentOccupation

        OPTIONAL {?parentOccupation rdfs:label ?parentOccupationlabel}    
          }
}
GROUP BY ?parentOccupation ?parentOccupationlabel 
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

SELECT ?parentOccupation ?parentOccupationlabel (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        {?s a wd:Q5.
        
        ## property path:
        ?s  wdt:P106/wdt:P279/wdt:P279  ?parentOccupation

        OPTIONAL {?parentOccupation rdfs:label ?parentOccupationlabel}    
          }
}
GROUP BY ?parentOccupation ?parentOccupationlabel 
ORDER BY DESC(?n)
LIMIT 20
````
### Third classification level
````sparql
### Third classification level

# no result after first insert
# result after first insert ! 



PREFIX franzOption_defaultDatasetBehavior: <franz:rdf>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?parentOccupation ?parentOccupationlabel (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        {?s a wd:Q5.
        
        ## property path:
        ?s  wdt:P106/wdt:P279/wdt:P279/wdt:P279  ?parentOccupation

        OPTIONAL {?parentOccupation rdfs:label ?parentOccupationlabel}    
          }
}
GROUP BY ?parentOccupation ?parentOccupationlabel 
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

SELECT ?parentOccupation ?parentOccupationlabel (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        {?s a wd:Q5.
        
        ## property path:
        ?s  wdt:P106 ?c1.
        ?c1 wdt:P279 ?c2.
        ?c2 wdt:P279 ?c3.
        ?c3 wdt:P279  ?parentOccupation
        ## Avoid recursivity
        FILTER ( ?c2 != ?c1)
        FILTER ( ?c3 not in (?c1, ?c2) )
        FILTER (  ?parentOccupation not in (?c1, ?c2, ?c3) )

        OPTIONAL {?parentOccupation rdfs:label ?parentOccupationlabel}    
          }
}
GROUP BY ?parentOccupation ?parentOccupationlabel 
ORDER BY DESC(?n)
LIMIT 20
````
### Fourth classification level
````sparql
### Fourth classification level

# no result after first insert
# result after first insert ! 



PREFIX franzOption_defaultDatasetBehavior: <franz:rdf>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?parentOccupation ?parentOccupationlabel (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        {?s a wd:Q5.
        
        ## property path:
        ?s  wdt:P106 ?c1.
        ?c1 wdt:P279 ?c2.
        ?c2 wdt:P279 ?c3.
        ?c3 wdt:P279 ?c4.
        ?c4 wdt:P279  ?parentOccupation
        FILTER ( ?c2 != ?c1)
        FILTER ( ?c3 not in (?c1, ?c2) )
        FILTER ( ?c4 not in (?c1, ?c2, ?c3) )
        FILTER (  ?parentOccupation not in (?c1, ?c2, ?c3, ?c4) )

        OPTIONAL {?parentOccupation rdfs:label ?parentOccupationlabel}    
          }
}
GROUP BY ?parentOccupation ?parentOccupationlabel 
ORDER BY DESC(?n)
LIMIT 20
````
## Inspect the instance of metaclasses to occupations
````sparql
## 

PREFIX franzOption_defaultDatasetBehavior: <franz:rdf>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

SELECT ?occupation ?occupationLabel ?metaclass ?metaclassLabel ?n
WHERE
    {
        GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        ## Find the persons in the imported graph
        {SELECT ?occupation ?occupationLabel (COUNT(*) as ?n)
        WHERE 
                {?item a wd:Q5.
                ?item wdt:P106 ?occupation.
                ?occupation rdfs:label ?occupationLabel}
        GROUP BY ?occupation ?occupationLabel      

        }
        ## 
        SERVICE <https://query.wikidata.org/sparql>
            {
                # instance of
                ?occupation wdt:P31 ?metaclass.
                BIND (?metaclassLabel as ?metaclassLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
            }
                
        }
ORDER BY DESC(?n)
LIMIT 30

````
## Inspect parent occupations fields
The aim is to find a way to distinguish between occupations
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
?occupation ?occupationLabel ?parentField ?parentFieldLabel
?n
          #?parentKnowledgeClassification ?parentKnowledgeClassificationLabel 
          # (SUM(?n) as ?sn)
WHERE {
    GRAPH <https://github.com/VericaD/nobel_laureates_pc/blob/main/graph/wikidata-imported-data.md>
        {
          SELECT ?occupation ?occupationLabel
          ?parentOccupation (COUNT(*) as ?n)
        
        WHERE
        {
          ?s a wd:Q5;  
              wdt:P106 ?occupation.
          ?occupation rdfs:label ?occupationLabel.
          ?occupation  wdt:P279  ?parentOccupation.
          }  
        GROUP BY ?occupation ?occupationLabel
          ?parentOccupation
        }

        SERVICE <https://query.wikidata.org/sparql>
          {
              # field of parent occupation / instance of /  subclass of  
              ?parentOccupation wdt:P425 / wdt:P31 ?parentField.
              # ?parentField wdt:P279 ?parentKnowledgeClassification.


              #BIND (?occupation as ?occupationLabel)
              BIND (?parentFieldLabel as ?parentFieldLabel)
              # BIND (?parentKnowledgeClassificationLabel as ?parentKnowledgeClassificationLabel)
              SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
          }

}
# GROUP BY ?occupation ?occupationLabel ?parentField ?parentFieldLabel
          # ?parentKnowledgeClassification ?parentKnowledgeClassificationLabel 
ORDER BY DESC(?n)
LIMIT 30
````
````sparql
### Create pairs of occupations 
# and add the birth year of the person

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

SELECT ?item ?birthYear ?occupation ?occupationLabel ?occupation_1 ?occupation_1Label
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
                ?item wdt:P106 ?occupation.
                ?item wdt:P106 ?occupation_1.
                FILTER (str(?occupationLabel) < str(?occupation_1Label))
                BIND (?occupationLabel as ?occupationLabel)
                BIND (?occupation_1Label as ?occupation_1Label)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
            }
                
        }


````

