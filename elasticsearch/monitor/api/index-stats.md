```
# curl 'http://127.0.0.1:9200/logstash-mweibo-2015.06.15/_stats/commit?level=shards&pretty'
...
  "indices" : {
    "logstash-2015.06.15" : {
      "primaries" : { },
      "total" : { },
      "shards" : {
        "0" : [ {
          "routing" : {
            "state" : "STARTED",
            "primary" : true,
            "node" : "AqaYWFQJRIK0ZydvVgASEw",
            "relocating_node" : null
          },
          "commit" : {
            "generation" : 726,
            "user_data" : {
              "translog_id" : "1434297603053",
              "sync_id" : "AU4LEh6wnBE6n0qcEXs5"
            },
            "num_docs" : 36792652
          }
        } ],
...
```


```
          "commit" : {
            "generation" : 590,
            "user_data" : {
              "translog_id" : "1434038402801"
            },
            "num_docs" : 29051938
          }
```
