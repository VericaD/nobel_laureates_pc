
```sparql
SELECT (COUNT(*) AS ?eff)
WHERE {
  ?item wdt:P31 wd:Q5;  # Any instance of a human (Q5 is for humans).
  wdt:P166 wd:Q38104 .  # Nobel Prize in Physics.
}


