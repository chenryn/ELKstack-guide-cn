# netflow

```
input {
    udp {
      port => 9995
      codec => netflow {
        definitions => "/home/administrator/logstash-1.4.2/lib/logstash/codecs/netflow/netflow.yaml"
        versions => [5]
      }
    }
  }

  output {
    stdout { codec => rubydebug }
    if ( [host] =~ "10\.1\.1[12]\.1" ) {
      elasticsearch {
        index => "logstash_netflow5-%{+YYYY.MM.dd}"
        host => "localhost"
      }
    } else {
      elasticsearch {
        index => "logstash-%{+YYYY.MM.dd}"
        host => "localhost"
      }
    }
  }
```


```
curl -XPUT localhost:9200/_template/logstash_netflow5 -d '{
    "template" : "logstash_netflow5-*",
    "settings": {
      "index.refresh_interval": "5s"
    },
    "mappings" : {
      "_default_" : {
        "_all" : {"enabled" : false},
        "properties" : {
          "@version": { "index": "analyzed", "type": "integer" },
          "@timestamp": { "index": "analyzed", "type": "date" },
          "netflow": {
            "dynamic": true,
            "type": "object",
            "properties": {
              "version": { "index": "analyzed", "type": "integer" },
              "flow_seq_num": { "index": "not_analyzed", "type": "long" },
              "engine_type": { "index": "not_analyzed", "type": "integer" },
              "engine_id": { "index": "not_analyzed", "type": "integer" },
              "sampling_algorithm": { "index": "not_analyzed", "type": "integer" },
              "sampling_interval": { "index": "not_analyzed", "type": "integer" },
              "flow_records": { "index": "not_analyzed", "type": "integer" },
              "ipv4_src_addr": { "index": "analyzed", "type": "ip" },
              "ipv4_dst_addr": { "index": "analyzed", "type": "ip" },
              "ipv4_next_hop": { "index": "analyzed", "type": "ip" },
              "input_snmp": { "index": "not_analyzed", "type": "long" },
              "output_snmp": { "index": "not_analyzed", "type": "long" },
              "in_pkts": { "index": "analyzed", "type": "long" },
              "in_bytes": { "index": "analyzed", "type": "long" },
              "first_switched": { "index": "not_analyzed", "type": "date" },
              "last_switched": { "index": "not_analyzed", "type": "date" },
              "l4_src_port": { "index": "analyzed", "type": "long" },
              "l4_dst_port": { "index": "analyzed", "type": "long" },
              "tcp_flags": { "index": "analyzed", "type": "integer" },
              "protocol": { "index": "analyzed", "type": "integer" },
              "src_tos": { "index": "analyzed", "type": "integer" },
              "src_as": { "index": "analyzed", "type": "integer" },
              "dst_as": { "index": "analyzed", "type": "integer" },
              "src_mask": { "index": "analyzed", "type": "integer" },
              "dst_mask": { "index": "analyzed", "type": "integer" }
            }
          }
        }
      }
    }
  }'
```
