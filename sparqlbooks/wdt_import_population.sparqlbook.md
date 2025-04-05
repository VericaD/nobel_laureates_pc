## Import to Allegrograph
First we check the basic properties of the population: name, sex, year of birth.
````sparql
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

SELECT DISTINCT ?item  ?itemLabel  ?gender ?year
        WHERE {
          { ?item wdt:P166 wd:Q38104 }  # Nobel Prize in Physics
          UNION
          { ?item wdt:P166 wd:Q44585 }  # Nobel Prize in Chemistry
          UNION
          { ?item wdt:P166 wd:Q80061 }  # Nobel Prize in Physiology or Medicine
          UNION
          { ?item wdt:P166 wd:Q37922 }  # Nobel Prize in Literature

          ?item wdt:P31 wd:Q5; # Must be a human
                wdt:P569 ?birthDate;
                wdt:P21 ?gender.       
                
        BIND(year(?birthDate) as ?year)
        #BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
        #FILTER(xsd:integer(?year) > 1750 && xsd:integer(?year) < 2001) 
        

        ## Add this clause in order to fill the variable      
        BIND ( ?itemLabel as ?itemLabel)
        SERVICE wikibase:label { bd:serviceParam wikibase:language "en" }   
        }
      
        LIMIT 10
    
````
### Preparing to import data
* This is a CONSTRUCT query, so instead of listing rows like SELECT, it builds a new RDF graph (triples).
````sparql
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

CONSTRUCT 
        {?item  rdfs:label ?itemLabel.
           ?item wdt:P21 ?gender.
           ?item wdt:P569 ?year.
           # ?item  wdt:P31 wd:Q5.
           # Noter qu'on odifie pour disposer de la propriété standard
           # pour déclarer l'appartenance d'une instance à une classe
           ?item  rdf:type wd:Q5. }
        
        WHERE {

        ## note the service address            
        SERVICE <https://query.wikidata.org/sparql>
            {
            { ?item wdt:P166 wd:Q38104 }  # Nobel Prize in Physics
            UNION
            { ?item wdt:P166 wd:Q44585 }  # Nobel Prize in Chemistry
            UNION
            { ?item wdt:P166 wd:Q80061 }  # Nobel Prize in Physiology or Medicine
            UNION
            { ?item wdt:P166 wd:Q37922 }  # Nobel Prize in Literature 
          
            ?item wdt:P31 wd:Q5;  # Any instance of a human.
                wdt:P569 ?birthDate;
                wdt:P21 ?gender.
        BIND(year(?birthDate) as ?year)
        #BIND(xsd:integer(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2")) AS ?year)
        FILTER(?year > 1750  && ?year < 2001)

        ## Add this clause in order to fill the variable      
        BIND ( ?itemLabel as ?itemLabel)
        SERVICE wikibase:label { bd:serviceParam wikibase:language "en" }   
        }
        }
        LIMIT 5
    
````

