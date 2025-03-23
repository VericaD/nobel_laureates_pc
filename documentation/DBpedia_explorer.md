Query that retrieves a list of Nobel Prize in Physics laureates and their birth
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
