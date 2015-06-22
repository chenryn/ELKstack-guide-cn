# logging

Elasticsearch emits a number of logs, which are placed in `$ES_HOME/logs`. The default logging level is INFO. It provides a moderate amount of information, but is designed to be rather light so that your logs are not enormous.

When debugging problems, particularly problems with node discovery (since this often depends on finicky network configurations), it can be helpful to bump up the logging level to DEBUG.

You could modify the logging.yml file and restart your nodes—but that is both tedious and leads to unnecessary downtime. Instead, you can update logging levels through the cluster-settings API that we just learned about.

To do so, take the logger you are interested in and prepend logger. to it. Let’s turn up the discovery logging:

```
PUT /_cluster/settings
{
    "transient" : {
        "logger.discovery" : "DEBUG"
    }
}
```

While this setting is in effect, Elasticsearch will begin to emit DEBUG-level logs for the discovery module.

Tip
Avoid TRACE. It is extremely verbose, to the point where the logs are no longer useful.

## Slowlog

There is another log called the slowlog. The purpose of this log is to catch queries and indexing requests that take over a certain threshold of time. It is useful for hunting down user-generated queries that are particularly slow.

By default, the slowlog is not enabled. It can be enabled by defining the action (query, fetch, or index), the level that you want the event logged at (WARN, DEBUG, and so forth) and a time threshold.

This is an index-level setting, which means it is applied to individual indices:

```
PUT /my_index/_settings
{
    "index.search.slowlog.threshold.query.warn" : "10s", 
    "index.search.slowlog.threshold.fetch.debug": "500ms", 
    "index.indexing.slowlog.threshold.index.info": "5s" 
}
```

You can also define these thresholds in your elasticsearch.yml file. Indices that do not have a threshold set will inherit whatever is configured in the static config.

Once the thresholds are set, you can toggle the logging level like any other logger:

```
PUT /_cluster/settings
{
    "transient" : {
        "logger.index.search.slowlog" : "DEBUG", 
        "logger.index.indexing.slowlog" : "WARN" 
    }
}
```
