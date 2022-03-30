# doo-payload-score
Score documents with payload in elasticsearch 7.9.2

document : https://ldh-6019.tistory.com/222?category=1029507/

## Releases
2022-03-28 `7.9.2` targets elasticsearch 7.9.2

## Overview


## Scoring
```java

```

## Plugin installation
Target elasticsearch version is 7.9.2 and java 1.8

**simple installation**
```shell
#docker 
COPY plugin/payload_score-7.9.2.zip plugin/payload_score-7.9.2.zip
RUN elasticsearch-plugin install file:///usr/share/elasticsearch/plugin/payload_score-7.9.2.zip

```



## Example

**mapping & setting**

```javascript 1.8
PUT paylaod_score_query
{
  "mappings": {
    "properties": {
      "color": {
        "type": "text",
        "term_vector": "with_positions_payloads",
        "analyzer": "payload_delimiter"
      }
    }
  },
  "settings": {
    "analysis": {
      "analyzer": {
        "payload_delimiter": {
          "tokenizer": "whitespace",
          "filter": [ "delimited_payload" ]
        }
      }
    }
  }
}

```

**test collection**
```javascript
POST paylaod_score_query/_doc/1
{
    "name" : "T-shirt S",
    "color" : "blue|1 green|2 yellow|3"
}

POST paylaod_score_query/_doc/2
{
    "name" : "T-shirt M",
    "color" : "blue|1 green|2 red|3"
}

POST paylaod_score_query/_doc/3
{
    "name" : "T-shirt XL",
    "color" : "blue|1 yellow|2"
}
```

**term 확인**
```bash
GET paylaod_score_query/_termvectors/1?fields=color
```

**query**
```bash
GET paylaod_score_query/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "t-shirt"
          }
        },
        {
          "span_or": {
            "clauses": [
              {
                "span_term": {
                  "color": "yellow"
                }
              }
            ]
          }
        }
      ]
    }
  }
}
```
