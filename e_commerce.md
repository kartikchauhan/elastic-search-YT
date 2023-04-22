```javascript

// Metrics aggregation

// Get sum of a field in all documents

GET e_commerce/_search
{
  "size": 0,
  "aggs": {
    "sum_unit_price": {
      "sum": {
        "field": "UnitPrice"
      }
    }
  }
}

// Result

"aggregations": {
    "sum_unit_price": {
      "value": 2498803.974
    }
  }

________________________________________________________________


// Get min value of a field in all documents

GET e_commerce/_search
{
  "size": 0,
  "aggs": {
    "min_unit_price": {
      "min": { // max for max value
        "field": "UnitPrice"
      }
    }
  }
}


________________________________________________________________


// Avg value of a field in all documents

GET e_commerce/_search
{
  "size": 0,
  "aggs": {
    "avg_unit_price": {
      "avg": {
        "field": "UnitPrice"
      }
    }
  }
}

________________________________________________________________

// All stats for a field

GET e_commerce/_search
{
  "size": 0,
  "aggs": {
    "all_stats_unit_price": {
      "stats": {
        "field": "UnitPrice"
      }
    }
  }
}

// Result

"aggregations": {
    "all_stats_unit_price": {
      "count": 541909,
      "min": -11062.06,
      "max": 38970,
      "avg": 4.611113626088513,
      "sum": 2498803.974
    }
  }

________________________________________________________________

// Return number of unique customers

GET e_commerce/_search
{
  "size": 0,
  "aggs": {
    "number_unique_customers": {
      "cardinality": {
        "field": "CustomerID"
      }
    }
  }
}

// Result

"aggregations": {
    "number_unique_customers": {
      "value": 4365
    }
  }

________________________________________________________________

// calculate the average unit price of items sold in Germany.

GET e_commerce/_search
{
  "size": 0,
  "query": {
    "term": {
      "Country": "Germany"
    }
  },
  "aggs": {
    "germany_avg_unit_price": {
      "avg": {
        "field": "UnitPrice"
      }
    }
  }
}

// Result

"aggregations": {
    "germany_avg_unit_price": {
      "value": 3.9669299631384938
    }
  }

________________________________________________________________

// Create a new index with own mapping

PUT e_commerce_2
{
  "mappings": {
    "properties": {
        "Country": {
          "type": "keyword"
        },
        "CustomerID": {
          "type": "long"
        },
        "Description": {
          "type": "text"
        },
        "InvoiceDate": {
          "type": "date",
          "format": "M/d/yyyy H:m"
        },
        "InvoiceNo": {
          "type": "keyword"
        },
        "Quantity": {
          "type": "long"
        },
        "StockCode": {
          "type": "keyword"
        },
        "UnitPrice": {
          "type": "double"
        }
      }  
  }
}

// Reindex

POST /_reindex
{
  "source": {
    "index": "e_commerce"
  },
  "dest": {
    "index": "e_commerce_2"
  }
}

________________________________________________________________

// Bucket Aggregation

// Date histogram aggregation with fixed interval

GET e_commerce_2/_search
{
  "aggs": {
    "by_8_hours": {
      "date_histogram": {
        "field": "InvoiceDate",
        "fixed_interval": "8h"
      }
    }
  }
}

// Result

"aggregations": {
    "by_8_hours": {
      "buckets": [
        {
          "key_as_string": "12/1/2010 8:0",
          "key": 1291190400000,
          "doc_count": 2301
        },
        {
          "key_as_string": "12/1/2010 16:0",
          "key": 1291219200000,
          "doc_count": 807
        },
        {
          "key_as_string": "12/2/2010 0:0",
          "key": 1291248000000,
          "doc_count": 10
        },

________________________________________________________________

// Date histogram with calendar interval ordered by Descending order of Date

GET e_commerce_2/_search
{
  "aggs": {
    "tx_by_month": {
      "date_histogram": {
        "field": "InvoiceDate",
        "calendar_interval": "1M",
        "order": {
          "_key": "desc"
        }
      }
    }
  }
}

// Result

"tx_by_month": {
      "buckets": [
        {
          "key_as_string": "12/1/2011 0:0",
          "key": 1322697600000,
          "doc_count": 25525
        },
        {
          "key_as_string": "11/1/2011 0:0",
          "key": 1320105600000,
          "doc_count": 84711
        },

________________________________________________________________

// The histogram aggregation creates buckets based on any numerical interval.

// Create buckets based on price interval that increases in increments of 10.

GET e_commerce_2/_search
{
  "size": 0, 
  "aggs": {
    "tx_per_price_interval": {
      "histogram": {
        "field": "UnitPrice",
        "interval": "10"
      }
    }
  }
}

// Result
"tx_per_price_interval": {
      "buckets": [
        {
          "key": -11070,
          "doc_count": 2
        },
        {
          "key": -11060,
          "doc_count": 0
        },
        {
          "key": -11050,
          "doc_count": 0
        },
```

```javascript

// Range Aggregation

// The range aggregation is similar to the histogram aggregation in that it can create buckets based on any numerical interval. The difference is that the range aggregation allows you to define intervals of varying sizes so you can customize it to your use case.

// Find number of transactions for items from varying price ranges(between 0 and $50, between $50-$200, and between $200 and up)?
GET e_commerce_2/_search
{
  "size": 0,
  "aggs": {
    "tx_per_custom_price_range": {
      "range": {
        "field": "UnitPrice",
        "ranges": [
          {
            "to": 50
          },
          {
            "from": 50,
            "to": 200
          },
          {
            "from": 200
          }
        ]
      }
    }
  }
}

// Result

  "aggregations": {
    "tx_per_custom_price_range": {
      "buckets": [
        {
          "key": "*-50.0",
          "to": 50,
          "doc_count": 540492
        },
        {
          "key": "50.0-200.0",
          "from": 50,
          "to": 200,
          "doc_count": 855
        },
        {
          "key": "200.0-*",
          "from": 200,
          "doc_count": 562
        }
      ]
    }
```

```javascript

// Terms Aggregation


// The terms aggregation creates a new bucket for every unique term it encounters for the specified field. It is often used to find the most frequently found terms in a document.

// identify 5 customers with the highest number of transactions(documents).

GET ${s}
{
  "size": 0,
  "aggs": {
    "top_5_customers": {
      "terms": {
        "field": "CustomerID",
        "size": 5
      }
    }
  }
}

// Result

"aggregations": {
    "top_5_customers": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 380391,
      "buckets": [
        {
          "key": 17841,
          "doc_count": 7983
        },
        {
          "key": 14911,
          "doc_count": 5903
        },
        {
          "key": 14096,
          "doc_count": 5128
        },
        {
          "key": 12748,
          "doc_count": 4642
        },
        {
          "key": 14606,
          "doc_count": 2782
        }
      ]

________________________________________________________________

// 5 customers with lowest number of transactions

GET ${s}
{
  "size": 0,
  "aggs": {
    "top_5_customers": {
      "terms": {
        "field": "CustomerID",
        "size": 5,
        "order": {
          "_count": "asc"
        }
      }
    }
  }
}

// Result 

"aggregations": {
    "top_5_customers": {
      "doc_count_error_upper_bound": -1,
      "sum_other_doc_count": 406824,
      "buckets": [
        {
          "key": 12503,
          "doc_count": 1
        },
        {
          "key": 12505,
          "doc_count": 1
        },
        {
          "key": 12943,
          "doc_count": 1
        },
        {
          "key": 13017,
          "doc_count": 1
        },
        {
          "key": 13099,
          "doc_count": 1
        }
      ]

________________________________________________________________

// Top 5 customers with highest doc_count in ascending order

GET ${s}
{
  "size": 0,
  "aggs": {
    "top_5_customers": {
      "terms": {
        "field": "CustomerID",
        "size": 5,
        "order": {
          "_count": "desc"
        }
      },
      "aggs": {
        "reversed_bucket": {
          "bucket_sort": {
            "sort": [
              {
                "_count": {
                  "order": "asc"
                }
              }
            ]
          }
        }
      }
    }
  }
}

// Result

"aggregations": {
    "top_5_customers": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 380391,
      "buckets": [
        {
          "key": 14606,
          "doc_count": 2782
        },
        {
          "key": 12748,
          "doc_count": 4642
        },
        {
          "key": 14096,
          "doc_count": 5128
        },
        {
          "key": 14911,
          "doc_count": 5903
        },
        {
          "key": 17841,
          "doc_count": 7983
        }
      ]
    }
  }

________________________________________________________________

// Combined Aggregation

// Calculate the sum of revenue per day.

GET ${s}
{
  "size": 0,
  "aggs": {
    "tx_per_day": {
      "date_histogram": {
        "field": "InvoiceDate",
        "calendar_interval": "day"
      },
      "aggs": {
        "revenue_per_day": {
          "sum": {
            "script": {
              "source": "doc['UnitPrice'].value * doc['Quantity'].value"
            }
          }
        }
      }
    }
  }
}

// Result 

"aggregations": {
    "tx_per_day": {
      "buckets": [
        {
          "key_as_string": "12/1/2010 0:0",
          "key": 1291161600000,
          "doc_count": 3108,
          "revenue_per_day": {
            "value": 58635.56
          }
        },
        {
          "key_as_string": "12/2/2010 0:0",
          "key": 1291248000000,
          "doc_count": 2109,
          "revenue_per_day": {
            "value": 46207.28
          }
        },

________________________________________________________________

// calculate the daily revenue and the number of unique customers per day in one go.

GET ${s}
{
  "size": 0,
  "aggs": {
    "tx_per_day": {
      "date_histogram": {
        "field": "InvoiceDate",
        "calendar_interval": "day"
      },
      "aggs": {
        "revenue_per_day": {
          "sum": {
            "script": {
              "source": "doc['UnitPrice'].value * doc['Quantity'].value"
            }
          }
        },
        "unique_customers_per_day": {
          "cardinality": {
            "field": "CustomerID"
          }
        }
      }
    }
  }
}

________________________________________________________________

// Sort the results on the basis of daily revenue

GET ${s}
{
  "size": 0,
  "aggs": {
    "tx_per_day": {
      "date_histogram": {
        "field": "InvoiceDate",
        "calendar_interval": "day",
        "order": {
          "revenue_per_day": "desc"
        }
      },
      "aggs": {
        "revenue_per_day": {
          "sum": {
            "script": {
              "source": "doc['UnitPrice'].value * doc['Quantity'].value"
            }
          }
        },
        "unique_customers_per_day": {
          "cardinality": {
            "field": "CustomerID"
          }
        }
      }
    }
  }
}

