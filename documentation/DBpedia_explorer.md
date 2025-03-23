Retrieves a list of Nobel Prize in Physics laureates and their birth
```sparql
PREFIX dbr: <http://dbpedia.org/resource/>
PREFIX dbo: <http://dbpedia.org/ontology/>

SELECT DISTINCT ?laureate ?birthDate
WHERE {
  ?laureate dbo:award dbr:Nobel_Prize_in_Physics .  # Nobel Prize in Physics
  OPTIONAL {
    ?laureate dbo:birthDate ?birthDate .   # Optional birth date
  }
}
```

* The number of Nobel Prize in Physics winners (in total 163)
  
  ```sparql
  PREFIX dbr: <http://dbpedia.org/resource/>
  PREFIX dbo: <http://dbpedia.org/ontology/>
  PREFIX dbp: <http://dbpedia.org/property/>
  
  SELECT (COUNT(*) as ?effectif)
  WHERE { 
      ?laureate dbo:award dbr:Nobel_Prize_in_Physics .
  }

  ```

Retrieves a list of Nobel Prize in Chemitry laureates and their birth
```sparql
  PREFIX dbr: <http://dbpedia.org/resource/>
  PREFIX dbo: <http://dbpedia.org/ontology/>

  SELECT DISTINCT ?laureate ?birthDate
  WHERE {
    ?laureate dbo:award dbr:Nobel_Prize_in_Chemistry .  # Nobel Prize in Chemitry 
    OPTIONAL {
      ?laureate dbo:birthDate ?birthDate .   # Optional birth date
    }
  }
```
* The number of Nobel Prize in Chemistry winners (in total 122)
  ```sparql
  PREFIX dbr: <http://dbpedia.org/resource/>
  PREFIX dbo: <http://dbpedia.org/ontology/>
  PREFIX dbp: <http://dbpedia.org/property/>
  
  SELECT (COUNT(*) as ?effectif)
  WHERE { 
      ?laureate dbo:award dbr:Nobel_Prize_in_Chemistry .
  }
