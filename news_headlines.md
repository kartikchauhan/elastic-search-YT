// Get count of all documents instead of default 10000

```js
GET news_headlines/_search
{
  "track_total_hits": true
}

// Result

"hits": {
    "total": {
      "value": 85000,
      "relation": "eq"
    },

________________________________________________________________

// In last 6 months

GET news_headlines/_search
{
  "query": {
    "range": {
      "date": {
        "gte": "now-6M",
        "lte": "now"
      }
    }
  }
}

________________________________________________________________

// In date range

GET news_headlines/_search
{
  "query": {
    "range": {
      "date": {
        "gte": "2022-09-23",
        "lte": "2023-09-23"
      }
    }
  }
}

________________________________________________________________

// Aggregation query

GET news_headlines/_search
{
  "aggs": {
    "by_category": {
      "terms": {
        "field": "category",
        "size": 2
      }
    }
  }
}

// Result

  "aggregations": {
    "by_category": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 46652,
      "buckets": [
        {
          "key": "POLITICS",
          "doc_count": 27866
        },
        {
          "key": "ENTERTAINMENT",
          "doc_count": 10482
        }
      ]
    }
  }

________________________________________________________________

// Combine query and aggregation
// Return results which are popular topics in ENTERTAINMENT category 

GET news_headlines/_search
{
  "query": {
    "match": {
      "category": "ENTERTAINMENT"
    }
  },
  "aggregations": {
    "popular_in_entertainment": {
      "significant_text": {
        "field": "headline"
      }
    }
  }
}

// Result

"aggregations": {
    "popular_in_entertainment": {
      "doc_count": 10482,
      "bg_count": 85000,
      "buckets": [
        {
          "key": "trailer",
          "doc_count": 251,
          "score": 0.13481367351103976,
          "bg_count": 307
        },
        {
          "key": "kardashian",
          "doc_count": 219,
          "score": 0.1287192939301251,
          "bg_count": 248
        },

// In these cases the words being selected are not simply the most popular terms in results. The most popular words tend to be very boring (and, of, the, we, I, they …​). The significant words are the ones that have undergone a significant change in popularity measured between a foreground and background set. If the term "H5N1" only exists in 5 documents in a 10 million document index and yet is found in 4 of the 100 documents that make up a user’s search results that is significant and probably very relevant to their search. 5/10,000,000 vs 4/100 is a big swing in frequency.

________________________________________________________________


// Following query uses "OR" logic. If a document contains one of the search terms, Elasticsearch will consider that document as a hit.

// High recall value

GET news_headlines/_search
{
  "query": {
    "match": {
      "headline": "Khloe Kardashian Kendall Jenner"
    }
  }
}

// Result

"hits": {
    "total": {
      "value": 442,
      "relation": "eq"

________________________________________________________________


// Increase precision

GET news_headlines/_search
{
  "query": {
    "match": {
      "headline": {
        "query": "Khloe Kardashian Kendall Jenner",
        "operator": "and"
      }
    }
  }
}

// Result 

"hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },

________________________________________________________________


// Fine tune precision and recall

GET news_headlines/_search
{
  "query": {
    "match": {
      "headline": {
        "query": "Khloe Kardashian Kendall Jenner",
        "minimum_should_match": 3
      }
    }
  }
}

// Result

"hits": {
    "total": {
      "value": 2,
      "relation": "eq"

________________________________________________________________

// Exact match for terms in the same order next to each other

GET news_headlines/_search
{
  "query": {
    "match_phrase": {
      "headline": {
        "query": "For All Americans"
      }
    }
  }
}

________________________________________________________________

// Search in multiple fields

GET news_headlines/_search
{
  "query": {
    "multi_match": {
      "query": "Michelle Obama",
      "fields": [
        "headline",
        "short_description",
        "authors"
        ]
    }
  }
}

________________________________________________________________

// Per-field boosting

// Boost a field

GET news_headlines/_search
{
  "query": {
    "multi_match": {
      "query": "Michelle Obama",
      "fields": [
        "headline^2",
        "short_description",
        "authors"
        ]
    }
  }
}

________________________________________________________________


// match_phrase in multi-fields

GET news_headlines/_search
{
  "query": {
    "multi_match": {
      "query": "party planning",
      "fields": [
        "headline^2",
        "short_description"
      ],
      "type": "phrase"
    }
  }
}

________________________________________________________________

// Form query to find political headlines about Michelle Obama published before the year 2016.

GET news_headlines/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match_phrase": {
            "headline": "Michelle Obama"  
          }
        },
        {
          "term": {
            "category": "POLITICS"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "date": {
              "lte": "2016",
              "format": "yyyy"
            }
          }
        }
      ]
    }
  }
}

________________________________________________________________

// The following query ask Elasticsearch to query all data that has the phrase "Michelle Obama" in the headline. Then, perform aggregations on the queried data and retrieve up to 100 categories that exist in the queried data.


GET news_headlines/_search
{
  "query": {
    "match_phrase": {
      "headline": "Michelle Obama"
    }
  },
  "aggregations": {
    "by_category": {
      "terms": {
        "field": "category",
        "size": 100
      }
    }
  }
}

_______________________________________________________________







```
