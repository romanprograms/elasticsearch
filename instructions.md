
```sh
curl -H "Content-Type: application/json" <URL> -d '<BODY>' ```
```sh
docker run -d \
  --name elasticsearch \
  -p 9200:9200 \
  -e "discovery.type=single-node" \
  docker.elastic.co/elasticsearch/elasticsearch:9.0.8
```
with security turned off
```sh
docker run -d \
  --name elasticsearch \
  -p 9200:9200 -p 9300:9300 \
  -e "discovery.type=single-node" \
  -e "xpack.security.enabled=false" \
  docker.elastic.co/elasticsearch/elasticsearch:8.15.2
```


```sh

docker inspect -f '{{json .NetworkSettings.Ports}}' elasticsearch | jq
docker inspect -f '{{json .State.Health}}' elasticsearch | jq
```

```sh
docker stop elasticsearch && docker rm elasticsearch

```


```sh
docker exec -it elasticsearch curl -s localhost:9200 | head
```

check elastic search running
```sh
curl -XGET 127.0.0.1:9200  

```

```sh
curl -H "Accept: application/json" http://media.sundog-soft.com/es9/shakes-mapping.json -o shakes-mapping.json
```

```sh

curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/shakespeare --data-binary @shakes-mapping.json
```


Download shakespeare data
```sh
curl -H "Accept: application/json" http://media.sundog-soft.com/es9/shakespeare_9.0.json -o shakespeare_9.0.json
```

Index data to Elastic Search

```sh

curl -H "Content-Type: application/json" -XPUT '127.0.0.1:9200/shakespeare/_bulk' --data-binary @shakespeare_9.0.json
```


```sh
curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/shakespeare/_search?pretty' -d '
    {
        "query": {
            "match_phrase": {
                "text-entry": "to be or not to be"

            }
    
        }
   }
'
```

```sh
curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies -d '
    {
      "mappings": {
        "properties": {
          "year": {
            "type": "date"
          }
        }
      }
    }
'
curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies/_doc/109487 -d '
    {
        "genre": ["IMAX", "Sci-Fi"],
        "title": "Interstellar",
        "year": 2014
    }
'

curl -X GET 127.0.0.1:9200/movies/_mapping
curl -XGET "127.0.0.1:9200/movies/_search?pretty"

```
```sh
curl -H "Content-Type: application/x-ndjson" -XPUT "127.0.0.1:9200/_bulk?pretty" --data-binary @movies.json
'

```


```sh
## Partial update
curl -H "Content-Type: application/json" -XPOST 127.0.0.1:9200/movies/_update/109487 -d '
    {
      "doc": {
        "title": "Interstellar"
      }
    }
'
#full update same as inserting a doc
curl -H "Content-Type: application/json" -XPUT "127.0.0.1:9200/movies/_doc/109487?pretty" -d '
    {
        "genre": ["IMAX", "Sci-Fi"],
        "title": "Interstellar foo",
        "year": 2014
    }
'
#search for this movie id 
curl -XGET "127.0.0.1:9200/movies/_doc/109487?pretty"

```


# Delete
```sh
# deleting by document id
curl -XDELETE "127.0.0.1:9200/movies/_doc/58559?pretty"

# search

curl -XGET "127.0.0.1:9200/movies/_search?pretty&q=Dark"


```
# Optimistic Concurrency Control
```sh
curl -H "Content-Type: application/json" -XPUT "127.0.0.1:9200/movies/_doc/109487?if_seq_no=7&if_primary_term=1" -d '
    {
        "genre": ["IMAX", "Sci-Fi"],
        "title": "Interstellar foo",
        "year": 2014
    }
'
``` 
if you repeat this you will get an error 
```json
{
  "error" : {
    "root_cause" : [
      {
        "type" : "version_conflict_engine_exception",
        "reason" : "[109487]: version conflict, required seqNo [7], primary term [1]. current document has seqNo [21] and primary term [1]",
        "index_uuid" : "OcB_WCDNRzG1E9hPY6JinA",
        "shard" : "0",
        "index" : "movies"
      }
    ],
    "type" : "version_conflict_engine_exception",
    "reason" : "[109487]: version conflict, required seqNo [7], primary term [1]. current document has seqNo [21] and primary term [1]",
    "index_uuid" : "OcB_WCDNRzG1E9hPY6JinA",
    "shard" : "0",
    "index" : "movies"
  },
  "status" : 409
}
```

```sh

curl -H "Content-Type: application/json" -XPOST "127.0.0.1:9200/movies/_update/109487?retry_on_conflicts=5" -d '
    {
      "doc": {
        "title": "Interstellar"
      }
    }
'
```
# Using analyzers and tokens


```sh
curl -H "Content-Type: application/json" -XGET "127.0.0.1:9200/movies/_search?pretty" -d '
  {
    "query": {
       "match": {
         "title": "Star Trek"
       }
    }
  }
'

curl -H "Content-Type: application/json" -XGET "127.0.0.1:9200/movies/_search?pretty" -d '
  {
    "query": {
       "match_phrase": {
         "genre": "sci"
       }
    }
  }
'
# Delete the entire index

curl -XDELETE "127.0.0.1:9200/movies"
```
# New mapping 
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
```
 # Data Modeling and Parent/Child Relationships, Part 2
```sh
# franchise is the parent 
# film is the child
curl -H "Content-Type: application/json" -XPUT "127.0.0.1:9200/series" -d '
{
  "mappings": {
    "properties": {
      "film_to_franchise": {
        "type": "join",
        "relations": { "franchise": "film" }
      }
    }
  }
}
'

curl -H "Content-Type: application/x-ndjson" -XPUT "127.0.0.1:9200/_bulk?pretty" --data-binary @series.json

# find all the films in the franchise
curl -H "Content-Type: application/json" -XGET "127.0.0.1:9200/series/_search?pretty" -d '
{
  "query": {
    "has_parent":  {
      "parent_type": "franchise",
      "query": {
         "match": {
           "title": "Star Wars"
         }
      }
    }
  }
}
'

# find all the films in the franchise
curl -H "Content-Type: application/json" -XGET "127.0.0.1:9200/series/_search?pretty" -d '
{
  "query": {
    "has_parent":  {
      "parent_type": "franchise",
      "query": {
         "match": {
           "title": "Star Wars"
         }
      }
    }
  }
}
'


curl -H "Content-Type: application/json" -XGET "127.0.0.1:9200/series/_search?pretty" -d '
{
  "query": {
    "has_child":  {
      "type": "film",
      "query": {
         "match": {
           "title": "The Force Awakens"
         }
      }
    }
  }
}
'
```

# Flattened Data Types
```sh
curl -XGET "127.0.0.1:9200/_cluster/state?pretty=true" >> es-cluster-state.json
```

1.
```sh

curl -H "Content-Type: application/json" -XPUT "http://127.0.0.1:9200/demo-default/_doc/1" -d'{
  "message": "[5592:1:0309/123054.737712:ERROR:child_process_sandbox_support_impl_linux.cc(79)] FontService unique font name matching request did not receive a response.",
  "fileset": {
    "name": "syslog"
  },
  "process": {
    "name": "org.gnome.Shell.desktop",
    "pid": 3383
  },
  "@timestamp": "2020-03-09T18:00:54.000+05:30",
  "host": {
    "hostname": "bionic",
    "name": "bionic"
  }
}'

# 2.

curl -H "Content-Type: application/json"   -XGET "http://127.0.0.1:9200/demo-default/_mapping?pretty=true"

# 3.

curl -H "Content-Type: application/json"  -XGET "http://127.0.0.1:9200/_cluster/state?pretty=true" >> es-cluster-state.json

# 4.

curl -H "Content-Type: application/json"  -XPUT "http://127.0.0.1:9200/demo-flattened"

# 5.

curl -H "Content-Type: application/json"  -XPUT "http://127.0.0.1:9200/demo-flattened/_mapping" -d'{
  "properties": {
    "host": {
      "type": "flattened"
    }
  }
}'

# 6.

curl -H "Content-Type: application/json"  -XPUT "http://127.0.0.1:9200/demo-flattened/_doc/1" -d'{
  "message": "[5592:1:0309/123054.737712:ERROR:child_process_sandbox_support_impl_linux.cc(79)] FontService unique font name matching request did not receive a response.",
  "fileset": {
    "name": "syslog"
  },
  "process": {
    "name": "org.gnome.Shell.desktop",
    "pid": 3383
  },
  "@timestamp": "2020-03-09T18:00:54.000+05:30",
  "host": {
    "hostname": "bionic",
    "name": "bionic"
  }
}'

# 7.

curl -H "Content-Type: application/json"  -XGET "http://127.0.0.1:9200/demo-flattened/_mapping?pretty=true"

# 8.

curl -H "Content-Type: application/json"  -XPOST "http://127.0.0.1:9200/demo-flattened/_update/1" -d'{
    "doc" : {
        "host" : {
          "osVersion": "Bionic Beaver",
          "osArchitecture":"x86_64"
        }
    }
}'

# 9.

curl -H "Content-Type: application/json"  -XGET "http://127.0.0.1:9200/demo-flattened/_search?pretty=true" -d'{
  "query": {
    "term": {
      "host": "Bionic Beaver"
    }
  }
}'

# 10.

curl -H "Content-Type: application/json"  -XGET "http://127.0.0.1:9200/demo-flattened/_search?pretty=true" -d'{
  "query": {
    "term": {
      "host.osVersion": "Bionic Beaver"
    }
  }
}'

# 11.

curl -H "Content-Type: application/json"  -XGET "http://127.0.0.1:9200/demo-flattened/_search?pretty=true" -d'{
  "query": {
    "term": {
      "host.osVersion": "Beaver"
    }
  }
}'
```
# Mapping Exceptions

```sh
# 1.
curl -H "Content-Type: application/json"   --request PUT 'http://localhost:9200/microservice-logs' \
--data-raw '{
   "mappings": {
       "properties": {
           "timestamp": { "type": "date"  },
           "service": { "type": "keyword" },
           "host_ip": { "type": "ip" },
           "port": { "type": "integer" },
           "message": { "type": "text" }
       }
   }
}'

```


2.

{"timestamp": "2020-04-11T12:34:56.789Z", "service": "ABC", "host_ip": "10.0.2.15", "port": 12345, "message": "Started!" }


```sh

# 3.

curl -H "Content-Type: application/json" --request POST 'http://localhost:9200/microservice-logs/_doc?pretty' \
--data-raw '{"timestamp": "2020-04-11T12:34:56.789Z", "service": "XYZ", "host_ip": "10.0.2.15", "port": "15000", "message": "Hello!" }'

```

```sh

# 4.

curl -H "Content-Type: application/json" --request POST 'http://localhost:9200/microservice-logs/_doc?pretty' \
--data-raw '{"timestamp": "2020-04-11T12:34:56.789Z", "service": "XYZ", "host_ip": "10.0.2.15", "port": "NONE", "message": "I am not well!" }'

```

```sh
# 5.
curl -H "Content-Type: application/json" --request POST 'http://localhost:9200/microservice-logs/_close'
 
curl -H "Content-Type: application/json" --location --request PUT 'http://localhost:9200/microservice-logs/_settings' \
--data-raw '{
   "index.mapping.ignore_malformed": true
}'
 
curl -H "Content-Type: application/json" --request POST 'http://localhost:9200/microservice-logs/_open'
```

```sh
6.

curl -H "Content-Type: application/json" --request POST 'http://localhost:9200/microservice-logs/_doc?pretty' \
--data-raw '{"timestamp": "2020-04-11T12:34:56.789Z", "service": "XYZ", "host_ip": "10.0.2.15", "port": "NONE", "message": "I am not well!" }'

```

```sh
# 7.

curl -H "Content-Type: application/json" --request POST 'http://localhost:9200/microservice-logs/_doc?pretty' \
--data-raw '{"timestamp": "2020-04-11T12:34:56.789Z", "service": "ABC", "host_ip": "10.0.2.15", "port": 12345, "message": {"data": {"received":"here"}}}'

```

```sh
8.

curl -H "Content-Type: application/json" --request POST 'http://localhost:9200/microservice-logs/_doc?pretty' \
--data-raw '{"timestamp": "2020-04-11T12:34:56.789Z", "service": "ABC", "host_ip": "10.0.2.15", "port": 12345, "message": "Received...", "payload": {"data": {"received":"here"}}}'

```

```sh
# 9.

curl -H "Content-Type: application/json" --request POST 'http://localhost:9200/microservice-logs/_doc?pretty' \
--data-raw '{"timestamp": "2020-04-11T12:34:56.789Z", "service": "ABC", "host_ip": "10.0.2.15", "port": 12345, "message": "Received...", "payload": {"data": {"received": {"even": "more"}}}}'
```
```sh
# 10.
thousandone_fields_json=$(echo {1..1001..1} | jq -Rn '( input | split(" ") ) as $nums | $nums[] | . as $key | [{key:($key|tostring),value:($key|tonumber)}] | from_entries' | jq -cs 'add')
echo "$thousandone_fields_json"

```

```sh
curl --location --request PUT 'http://localhost:9200/big-objects'
curl -H "Content-Type: application/json"  --request POST 'http://localhost:9200/big-objects/_doc?pretty' --data-raw "$thousandone_fields_json"

``` 
```sh
# changing fields limit
curl -H "Content-Type: application/json"  --location --request PUT   'http://localhost:9200/big-objects/_settings' --data-raw '
{
  "index.mapping.total.fields.limit": 1001
}
'
```
