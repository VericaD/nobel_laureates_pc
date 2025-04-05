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

