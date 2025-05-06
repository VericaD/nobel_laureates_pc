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
## Import data
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
````
````sparql
````
````sparql
````
````sparql
````
````sparql
````
