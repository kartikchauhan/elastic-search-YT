PUT /candy

POST candy/_doc
{
  "first_name": "Kartik",
  "candy": "Melody"
}

PUT candy/_doc/1
{
  "first_name": "Chauhan",
  "candy": "rolos"
}

GET candy/_doc/1

PUT candy/_doc/1 // Updates existing document
{
  "first_name": "Chauhan",
  "candy": "rolos 2"
}

// To prevent update

PUT candy/_create/1
{
  "first_name": "Chauhan",
  "candy": "rolos 2"
}

// Result

{
  "error": {
    "root_cause": [
      {
        "type": "version_conflict_engine_exception",
        "reason": "[1]: version conflict, document already exists (current version [3])",
        "index_uuid": "-D-sUzIMTmS0mAOmq_PtKg",
        "shard": "0",
        "index": "candy"
      }
    ],
    "type": "version_conflict_engine_exception",
    "reason": "[1]: version conflict, document already exists (current version [3])",
    "index_uuid": "-D-sUzIMTmS0mAOmq_PtKg",
    "shard": "0",
    "index": "candy"
  },
  "status": 409
}

// Update a single field

POST candy/_update/1
{
  "doc": {
    "candy": "rolos 3"  
  } 
}

