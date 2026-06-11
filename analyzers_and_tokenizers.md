Sometimes text fields should be exact match
- Use keyword mapping instead of text


Search on analyzed text fields will return anything remotely relavant
 - Depending on the analyzer, results will be case-insensitive,
 stemmed, stopwords removed, synonyms applied, etc.
 - Searches with multiple terms need not match them all.


 ```sh
 curl -H "Content-Type: application/json" -XGET http://localhost:9200/movies/_search?pretty  -d '
    {
      "query": {
          "match": {
              "title": "Star Trek"
           }
        }
    }
 '
 ```
```json
{
  "query": {
    "match_phrase": {
       "genre": "Sci"
    }
  }
}
```
```sh
curl -H "Content-Type: application/json" -XGET http://localhost:9200/movies/_search?pretty  -d '
{
  "query": {
    "match_phrase": {
       "genre": "Sci"
    }
  }
}
'
```

curl -XDELETE http://localhost:9200/movies

```sh
curl -H "Content-Type: application/json" -XPUT http://localhost:9200/movies -d '
{
    "mappings": {
        "properties": {
            "id": {"type": "integer"},
            "year": {"type": "date"},
            "genre": {"type": "keyword"},
            "title": {"type": "text", "analyzer": "english"}
        }
    }
}
'
```

```sh

curl -H "Content-Type: application/json" -XPUT http://localhost:9200/_bulk?pretty --data-binary @movies.json
```
