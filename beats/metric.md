# metricbeat

使用 beat 监控服务性能指标是 ElasticStack 一个常见的使用场景。2.x 时代要求用户对每类常见都需要单独开发自己的 xxxbeat 工具，然后各自编译使用。于是 Elastic.co 公司最终干脆把这件事情统一成了 metricbeat。

目前 metricbeat 支持以下服务性能指标：

* Apache
* HAProxy
* MongoDB
* MySQL
* Nginx
* PostgreSQL
* Redis
* System
* Zookeeper

## 配置示例

```yaml
metricbeat.modules:
    - module: system
      metricsets:
          - cpu
          - filesystem
          - memory
          - network
          - process
      enabled: true
      period: 10s
      processes: ['.*']
      cpu_ticks: false
    - module: apache
      metricsets: ["status"]
      enabled: true
      period: 1s
      hosts: ["http://127.0.0.1"]
```

## Apache

Apache 模块支持 2.2.31 以上的 2.2 系列，或 2.4.16 以上的 2.4 系列版本。

使用该模块要求被监控的 Apache 服务器上安装配置有 `mod_status` 扩展。通过该扩展可以监控到的 status 数据示例如下：

```json
    "apache": {
        "status": {
            "bytes_per_request": 1024,
            "bytes_per_sec": 0.201113,
            "connections": {
                "async": {
                    "closing": 0,
                    "keep_alive": 0,
                    "writing": 0
                },
                "total": 0
            },
            "cpu": {
                "children_system": 0,
                "children_user": 0,
                "load": 0.00652482,
                "system": 1.46,
                "user": 1.53
            },
            "hostname": "apache",
            "load": {
                "1": 0.55,
                "15": 0.31,
                "5": 0.31
            },
            "requests_per_sec": 0.000196399,
            "scoreboard": {
                "closing_connection": 0,
                "dns_lookup": 0,
                "gracefully_finishing": 0,
                "idle_cleanup": 0,
                "keepalive": 0,
                "logging": 0,
                "open_slot": 325,
                "reading_request": 0,
                "sending_reply": 1,
                "starting_up": 0,
                "total": 400,
                "waiting_for_connection": 74
            },
            "total_accesses": 9,
            "total_kbytes": 9,
            "uptime": {
                "server_uptime": 45825,
                "uptime": 45825
            },
            "workers": {
                "busy": 1,
                "idle": 74
            }
        }
    }
```

模块携带有一个预一定好的仪表盘，效果如下：
![](https://www.elastic.co/guide/en/beats/metricbeat/current/images/apache_httpd_server_status.png)

## HAProxy

HAProxy 模块支持 HAProxy 服务器 1.6 版本。

使用该模块要求在 HAProxy 服务器配置的 `global` 或 `default` 区域写有如下配置：

```
stats socket 127.0.0.1:14567
```

模块可以采集两类信息：info 和 stat。

其中 info 的返回数据示例如下：

```
    "haproxy": {
        "info": {
            "compress_bps_in": 0,
            "compress_bps_out": 0,
            "compress_bps_rate_limit": 0,
            "conn_rate": 0,
            "conn_rate_limit": 0,
            "cum_conns": 67,
            "cum_req": 67,
            "cum_ssl_conns": 0,
            "curr_conns": 0,
            "curr_ssl_conns": 0,
            "hard_max_conn": 4000,
            "idle_pct": 100,
            "max_conn": 4000,
            "max_conn_rate": 5,
            "max_pipes": 0,
            "max_sess_rate": 5,
            "max_sock": 8033,
            "max_ssl_conns": 0,
            "max_ssl_rate": 0,
            "max_zlib_mem_usage": 0,
            "mem_max_mb": 0,
            "nb_proc": 1,
            "pid": 53858,
            "pipes_free": 0,
            "pipes_used": 0,
            "process_num": 1,
            "run_queue": 2,
            "sess_rate": 0,
            "sess_rate_limit": 0,
            "ssl_babckend_key_rate": 0,
            "ssl_backend_max_key_rate": 0,
            "ssl_cache_misses": 0,
            "ssl_cached_lookups": 0,
            "ssl_frontend_key_rate": 0,
            "ssl_frontend_max_key_rate": 0,
            "ssl_frontend_session_reuse_pct": 0,
            "ssl_rate": 0,
            "ssl_rate_limit": 0,
            "tasks": 7,
            "ulimit_n": 8033,
            "uptime_sec": 13700,
            "zlib_mem_usage": 0
        }
    },
```

stat 的返回数据示例如下：

```
    "haproxy": {
        "stat": {
            "act": 1,
            "bck": 0,
            "bin": 0,
            "bout": 0,
            "check_duration": 0,
            "check_status": "L4CON",
            "chkdown": 1,
            "chkfail": 1,
            "cli_abrt": 0,
            "ctime": 0,
            "downtime": 13700,
            "dresp": 0,
            "econ": 0,
            "eresp": 0,
            "hanafail": 0,
            "hrsp_1xx": 0,
            "hrsp_2xx": 0,
            "hrsp_3xx": 0,
            "hrsp_4xx": 0,
            "hrsp_5xx": 0,
            "hrsp_other": 0,
            "iid": 3,
            "last_chk": "Connection refused",
            "lastchg": 13700,
            "lastsess": -1,
            "lbtot": 0,
            "pid": 1,
            "qcur": 0,
            "qmax": 0,
            "qtime": 0,
            "rate": 0,
            "rate_max": 0,
            "rtime": 0,
            "scur": 0,
            "sid": 1,
            "smax": 0,
            "srv_abrt": 0,
            "status": "DOWN",
            "stot": 0,
            "svname": "log1",
            "ttime": 0,
            "weight": 1,
            "wredis": 0,
            "wretr": 0
        }
   }
```

对这些 stat 数据名称有疑惑的，可以查阅 <http://www.haproxy.org/download/1.6/doc/management.txt> 文档。

## MongoDB

该模块支持 MongoDB 2.8 及以上版本。

```
"mongodb": {
    "status": {
        "asserts": {
            "msg": 0,
            "regular": 0,
            "rollovers": 0,
            "user": 0,
            "warning": 0
        },
        "background_flushing": {
            "average": {
                "ms": 16
            },
            "flushes": 37,
            "last": {
                "ms": 18
            },
            "last_finished": "2016-09-06T07:32:58.228Z",
            "total": {
                "ms": 624
            }
        },
        "connections": {
            "available": 838859,
            "current": 1,
            "total_created": 10
        },
        "extra_info": {
            "heap_usage": {
                "bytes": 62895448
            },
            "page_faults": 71
        },
        "journaling": {
            "commits": 1,
            "commits_in_write_lock": 0,
            "compression": 0,
            "early_commits": 0,
            "journaled": {
                "mb": 0
            },
            "times": {
                "commits": {
                    "ms": 0
                },
                "commits_in_write_lock": {
                    "ms": 0
                },
                "dt": {
                    "ms": 0
                },
                "prep_log_buffer": {
                    "ms": 0
                },
                "remap_private_view": {
                    "ms": 0
                },
                "write_to_data_files": {
                    "ms": 0
                },
                "write_to_journal": {
                    "ms": 0
                }
            },
            "write_to_data_files": {
                "mb": 0
            }
        },
        "local_time": "2016-09-06T07:33:15.546Z",
        "memory": {
            "bits": 64,
            "mapped": {
                "mb": 80
            },
            "mapped_with_journal": {
                "mb": 160
            },
            "resident": {
                "mb": 57
            },
            "virtual": {
                "mb": 356
            }
        },
        "network": {
            "in": {
                "bytes": 2258
            },
            "out": {
                "bytes": 93486
            },
            "requests": 39
        },
        "opcounters": {
            "command": 40,
            "delete": 0,
            "getmore": 0,
            "insert": 0,
            "query": 1,
            "update": 0
        },
        "opcounters_replicated": {
            "command": 0,
            "delete": 0,
            "getmore": 0,
            "insert": 0,
            "query": 0,
            "update": 0
        },
        "storage_engine": {
            "name": "mmapv1"
        },
        "uptime": {
            "ms": 45828938
        },
        "version": "3.0.12",
        "write_backs_queued": false
    }
}
```

## MySQL

该模块支持 MySQL 5.7.0 及以上版本。

```
"mysql": {
    "status": {
        "aborted": {
            "clients": 13,
            "connects": 16
        },
        "binlog": {
            "cache": {
                "disk_use": 0,
                "use": 0
            }
        },
        "bytes": {
            "received": 2100,
            "sent": 92281
        },
        "connections": 33,
        "created": {
            "tmp": {
                "disk_tables": 0,
                "files": 6,
                "tables": 0
            }
        },
        "delayed": {
            "errors": 0,
            "insert_threads": 0,
            "writes": 0
        },
        "flush_commands": 1,
        "max_used_connections": 2,
        "open": {
            "files": 14,
            "streams": 0,
            "tables": 106
        },
        "opened_tables": 113
    }
}
```

## Nginx

该模块支持 Nginx 1.9 及以上版本。并要求安装有 `mod_stub_status` 模块。

```
"nginx": {
    "stubstatus": {
        "accepts": 22,
        "active": 1,
        "current": 10,
        "dropped": 0,
        "handled": 22,
        "hostname": "nginx",
        "reading": 0,
        "requests": 10,
        "waiting": 0,
        "writing": 1
    }
}
```

## PostgreSQL

该模块支持 PostgreSQL 9 及以上版本。可以采集 activity，bgwriter 和 database 三类数据。

activity 示例数据如下：

```
"postgresql": {
    "activity": {
        "application_name": "",
        "backend_start": "2016-09-06T07:33:18.323Z",
        "client": {
            "address": "172.17.0.14",
            "hostname": "",
            "port": 57436
        },
        "database": {
            "name": "postgres",
            "oid": 12379
        },
        "pid": 162,
        "query": "SELECT * FROM pg_stat_activity",
        "query_start": "2016-09-06T07:33:18.325Z",
        "state": "active",
        "state_change": "2016-09-06T07:33:18.325Z",
        "transaction_start": "2016-09-06T07:33:18.325Z",
        "user": {
            "id": 10,
            "name": "postgres"
        },
        "waiting": false
    },
```

bgwriter 示例数据如下：

```
    "bgwriter": {
        "buffers": {
            "allocated": 191,
            "backend": 0,
            "backend_fsync": 0,
            "checkpoints": 0,
            "clean": 0,
            "clean_full": 0
        },
        "checkpoints": {
            "requested": 0,
            "scheduled": 7,
            "times": {
                "sync": {
                    "ms": 0
                },
                "write": {
                    "ms": 0
                }
            }
        },
        "stats_reset": "2016-09-05T18:49:53.575Z"
    },
```

database 示例数据如下：

```
    "database": {
        "blocks": {
            "hit": 0,
            "read": 0,
            "time": {
                "read": {
                    "ms": 0
                },
                "write": {
                    "ms": 0
                }
            }
        },
        "conflicts": 0,
        "deadlocks": 0,
        "name": "template1",
        "number_of_backends": 0,
        "oid": 1,
        "rows": {
            "deleted": 0,
            "fetched": 0,
            "inserted": 0,
            "returned": 0,
            "updated": 0
        },
        "temporary": {
            "bytes": 0,
            "files": 0
        },
        "transactions": {
            "commit": 0,
            "rollback": 0
        }
    }
```

## Redis

该模块支持 Redis 3 及以上版本。可以采集 info 和 keyspace 两类数据。

info 示例数据如下：

```
"redis": {
    "info": {
        "clients": {
            "biggest_input_buf": 0,
            "blocked": 0,
            "connected": 2,
            "longest_output_list": 0
        },
        "cluster": {
            "enabled": false
        },
        "cpu": {
            "used": {
                "sys": 0.33,
                "sys_children": 0,
                "user": 0.39,
                "user_children": 0
            }
        },
        "memory": {
            "allocator": "jemalloc-4.0.3",
            "used": {
                "lua": 37888,
                "peak": 883992,
                "rss": 4030464,
                "value": 883032
            }
        },
        "persistence": {
            "aof": {
                "bgrewrite": {
                    "last_status": "ok"
                },
                "enabled": false,
                "rewrite": {
                    "current_time": {
                        "sec": -1
                    },
                    "in_progress": false,
                    "last_time": {
                        "sec": -1
                    },
                    "scheduled": false
                },
                "write": {
                    "last_status": "ok"
                }
            },
            "loading": false,
            "rdb": {
                "bgsave": {
                    "current_time": {
                        "sec": -1
                    },
                    "in_progress": false,
                    "last_status": "ok",
                    "last_time": {
                        "sec": -1
                    }
                },
                "last_save": {
                    "changes_since": 2,
                    "time": 1475698251
                }
            }
        },
        "replication": {
            "backlog": {
                "active": 0,
                "first_byte_offset": 0,
                "histlen": 0,
                "size": 1048576
            },
            "connected_slaves": 0,
            "master_offset": 0,
            "role": "master"
        },
        "server": {
            "arch_bits": "64",
            "build_id": "5575d747b4b3b12c",
            "config_file": "",
            "gcc_version": "4.9.2",
            "git_dirty": "0",
            "git_sha1": "00000000",
            "hz": 10,
            "lru_clock": 16080842,
            "mode": "standalone",
            "multiplexing_api": "epoll",
            "os": "Linux 4.4.22-moby x86_64",
            "process_id": 1,
            "run_id": "d37e5ebfe0ae6c4972dbe9f0174a1637bb8247f6",
            "tcp_port": 6379,
            "uptime": 383,
            "version": "3.2.4"
        },
        "stats": {
            "commands_processed": 70,
            "connections": {
                "received": 17,
                "rejected": 0
            },
            "instantaneous": {
                "input_kbps": 0.07,
                "ops_per_sec": 2,
                "output_kbps": 0.07
            },
            "keys": {
                "evicted": 0,
                "expired": 0
            },
            "keyspace": {
                "hits": 0,
                "misses": 0
            },
            "latest_fork_usec": 0,
            "migrate_cached_sockets": 0,
            "net": {
                "input": {
                    "bytes": 1949
                },
                "output": {
                    "bytes": 4956554
                }
            },
            "pubsub": {
                "channels": 0,
                "patterns": 0
            },
            "sync": {
                "full": 0,
                "partial": {
                    "err": 0,
                    "ok": 0
                }
            }
        }
    },
```

keyspace 示例数据如下：

```
    "keyspace": {
        "avg_ttl": 0,
        "expires": 0,
        "id": "db0",
        "keys": 1
    }
}
```

## System

System 就是过去的 TopBeat，可以采集 core、cpu、diskio、filesystem、fsstat、load、memory、network 和 process 指标。这都是运维人员最熟悉的部分，就不再单独贴指标名称和示例了。

模块自带有一个预定义仪表盘，示例如下：

![](https://www.elastic.co/guide/en/beats/metricbeat/current/images/metricbeat_system_dashboard.png)

## Zookeeper

该模块支持 Zookeeper 3.4.0 及以上版本。采集的 mntr 数据示例如下：

```
"zookeeper": {
    "mntr": {
        "approximate_data_size": 27,
        "ephemerals_count": 0,
        "latency": {
            "avg": 0,
            "max": 0,
            "min": 0
        },
        "num_alive_connections": 1,
        "outstanding_requests": 0,
        "packets": {
            "received": 10,
            "sent": 9
        },
        "server_state": "standalone",
        "version": "3.4.8--1, built on 02/06/2016 03:18 GMT",
        "watch_count": 0,
        "znode_count": 4
    }
}
```

## docker 中的采集方式

metricbeat 的 system 数据大多采集自 /proc。而 docker 中，每个容器的实际数据是放在 /hostfs 而不是 /proc 里的。所以如果要用 metricbeat 采集容器数据，需要先挂载好对应路径：

```
$ sudo docker run \
  --volume=/proc:/hostfs/proc:ro \ 
  --volume=/sys/fs/cgroup:/hostfs/sys/fs/cgroup:ro \ 
  --volume=/:/hostfs:ro \ 
  --net=host 
  my/metricbeat:latest -system.hostfs=/hostfs
```
