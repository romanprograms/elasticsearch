```sh
curl -H "Content-Type: application/json" -XPUT "127.0.0.1:9200/movies?pretty" -d '
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
curl -H "Content-Type: application/x-ndjson" -XPUT "127.0.0.1:9200/_bulk?pretty" --data-binary @movies.json
'
```
# JSON Search In-Depth
```sh
curl -H 'Content-Type: application/json' -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
  "query": {
    "match": {
      "title": "star"
     }
  }
}
'
```
## Queries and Filters

Filters ask a yes/no question of your data
Queries return data in terms of relevance

Use filters when you can - they are faster and cacheable


```sh
# bool means this query must have trek within a title and range filter that contains a year
# greater than 2010
curl -H 'Content-Type: application/json' -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
  "query": {
    "bool": {
       "must": {"term": {"title: "trek}},
       "filter": {"range": {"year": {"gte: 2010"}}}
     }
  }
}
'
```
## Some types of filters
`Term`: Filter by exact values 
{"term": {"year": 2014}}

`Terms`: Match if any exact values in a list match
{"terms": {"genre": ["Sci-Fi", "Adventure"]}}


`Range`: Find numbers of dates in a given range (gt, gte, lt, lte)
{"range": {"year": {"gte": 2010}}}

`Exists`: Find documents where a field exists
{"exists": {"field": "tags"}}

`Missing`: Find documents where a field is misssing
{"missing": {"field": "tags"}}

`Bool`: Combine filters with Boolean logic (must, must_not, should)



## Some Types of Queries 
`Match_all` Returns all documents and is default. Normally used with a filter.
{"match_all": {}}


`Match`: Searches analyzed results, such as full text search.
{"match": {"title": "star"}}

`Multi_match`: Run the same query on multiple fields.
{"multi_match": {"query": "star", "fields": ["title", "synopsis"]}}

`Bool`: Works like a bool filter, but results are scored by relevance.


Queries are wrapped in "query": {} block
Filters are warpped in "filter": {} block.

You can combine filters inside queries, or queries inside filters too. 

```sh
curl -H "Content-Type: application/json" -XGET "127.0.0.1:9200/movies/_search?pretty" -d '
{
  "query": {
    "bool": {
      "must": {"term": {"title": "trek"}},
      "filter": {"range": {"year": {"gte": 2010}}}
    }
  }
}
'
```


```sh
curl -H "Content-Type: application/json" -XGET "127.0.0.1:9200/movies/_search?pretty" -d '
{
  "query": {
    "match": {
      "title" : "star"
    }
  }
}
'

```


```sh
curl -H "Content-Type: application/json" -XGET "127.0.0.1:9200/movies/_search?pretty" -d '
{
  "query": {
    "bool": {
      "must": {"term": {"title": "trek"}},
      "filter": {"range": {"year": {"gte": 2010}}}
    }
  }
}
'
```


# Phrase Matching

```sh
curl -H "Content-Type: application/json" -XGET "127.0.0.1:9200/movies/_search?pretty" -d '
{
  "query": {
    "match_phrase": {
       "title": "star wars"
    }
  }
}
'
```
## Slop
Order matters, but you're OK with some words being in between the terms:

```sh
curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
  "query": {
    "match_phrase": {
      "title": {"query": "star beyond", "slop": 1}
    }
  }
}
'
```
The slop represents how far you're willing to let a term move to satisfy a phrase (in either direction!)
Another example: "quick brown fox" would match "quick fox" with a slop of 1.


Remeber this is a query - results are sorted by relevance

Just use a really high slop if you want to get any documents that contain the worlds in your 
phrase, but want documents that have the words close together scored higher.
```sh
curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
  "query": {
    "match_phrase": {
      "title": {"query": "star beyond", "slop": 100}
    }
  }
}
'


```sh
curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
  "query": {
    "match": {
      "title": "Star Wars"
    }
  }
}
'

```sh
curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
  "query": {
    "match_phrase": {
      "title": "Star Wars"
    }
  }
}
'
```

```sh
curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
  "query": {
    "match_phrase": {
      "title": {"query": "star beyond", "slop": 1}
    }
  }
}
'
```


```sh
curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
  "query": {
    "match_phrase": {
      "title": {"query": "star beyond", "slop": 100}
    }
  }
}
'
```
```sh
curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
  "query": {
    "bool": {
      "must": {"match_phrase": {"title": "star wars"}},
      "filter": {"range": {"year": {"gte": 1980}}}
    }
  }
}
'
```

# Pagination
```sh
curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
  "from": 2,
  "size": 2,
  "query": {
    "match": {
      "genre": "Sci-Fi"
    }
  }
}
'
```
Deep pagination can kill performance.
Every result must be retrieved, collected, and sorted.
Enforce an upper bound on how many results you'll return to users.


# Sorting

```sh
curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
  "sort": "year"
}
'
`
```

A string field that is analyzed for full text search can't be used to sort documents

That is because it exists in the inverted index as individual terms, not as the entire string.

```sh
curl -XPUT "127.0.0.1:9200/movies/" -d '
{
  "mappings": {
    "properties": {
      "title": {
         "type": "text",
         "fields": {
            "raw": {
              {
                "type": "keyword"
              }
            }
          }
       }
    }
  }
}
'

```

Sadly, you cannot change the mapping on an existing index.
You'd have to delete it, set up a new mapping, and re-index it.
Like number of shards, this is something you should think about before importing data into your index.
```sh
curl  -XDELETE 127.0.0.1:9200/movies
curl -H "Content-Type:application/json" -XPUT "127.0.0.1:9200/movies" -d '
{
  "mappings": {
    "properties": {
      "title": {
         "type": "text",
         "fields": {
            "raw": {
                "type": "keyword"
              }
          }
      }
    }
  }
}
'

curl -H "Content-Type: application/x-ndjson" -XPUT "127.0.0.1:9200/_bulk?pretty" --data-binary @movies.json
curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
  "sort": "title.raw"
}
'
```
