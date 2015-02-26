You can set up Kibana and start exploring your Elasticsearch indices in minutes. All you need is:

* Elasticsearch 1.4.4 or later
* A modern web browser - [Supported Browsers](http://www.elasticsearch.com/support/matrix?_ga=1.149082614.1575542547.1409213558).
* Information about your Elasticsearch installation:
  * URL of the Elasticsearch instance you want to connect to.
  * Which Elasticsearch indices you want to search.

> If your Elasticsearch installation is protected by [Shield](http://www.elasticsearch.org/overview/shield/) see [Shield with Kibana 4](https://www.elasticsearch.org/guide/en/shield/current/_shield_with_kibana_4.html) for additional setup instructions.

## install and start kibanaedit

To get Kibana up and running:

1. Download the [Kibana 4 binary package](http://www.elasticsearch.org/overview/kibana/installation/) for your platform.
2. Extract the `.zip` or `tar.gz` archive file.
3. Run Kibana from the install directory: `bin/kibana` (Linux/MacOSX) or `bin\kibana.bat` (Windows).

That’s it! Kibana is now running on port 5601.

## connect kibana with elasticsearchedit

Before you can start using Kibana, you need to tell it which Elasticsearch indices you want to explore. The first time you access Kibana, you are prompted to define an *index pattern* that matches the name of one or more of your indices. That’s it. That’s all you need to configure to start using Kibana. You can add index patterns at any time from the [Settings tab](http://www.elasticsearch.org/guide/en/kibana/current/settings.html#settings-create-pattern).

> By default, Kibana connects to the Elasticsearch instance running on `localhost`. To connect to a different Elasticsearch instance, modify the Elasticsearch URL in the `kibana.yml` configuration file and restart Kibana. For information about using Kibana with your production nodes, see [Using Kibana in a Production Environment](./production.md).

To configure the Elasticsearch indices you want to access with Kibana:

1. Point your browser at port 5601 to access the Kibana UI. For example, `localhost:5601` or `http://YOURDOMAIN.com:5601`.
![](http://www.elasticsearch.org/guide/en/kibana/current/images/Start-Page.jpg)
2. Specify an index pattern that matches the name of one or more of your Elasticsearch indices. By default, Kibana guesses that you’re you’re working with data being fed into Elasticsearch by Logstash. If that’s the case, you can use the default `logstash-*` as your index pattern. The asterisk (*) matches zero or more characters in an index’s name. If your Elasticsearch indices follow some other naming convention, enter an appropriate pattern. The "pattern" can also simply be the name of a single index.
3. Select the index field that contains the timestamp that you want to use to perform time-based comparisons. Kibana reads the index mapping to list all of the fields that contain a timestamp. If your index doesn’t have time-based data, disable the `Index contains time-based events` option.
4. If new indices are generated periodically and have a timestamp appended to the name, select the `Use event times to create index names` option and select the `Index pattern interval`. This improves search performance by enabling Kibana to search only those indices that could contain data in the time range you specify. This is primarily applicable if you are using Logstash to feed data into Elasticsearch.
5. Click `Create` to add the index pattern. This first pattern is automatically configured as the default. When you have more than one index pattern, you can designate which one to use as the default from `Settings > Indices`.

Voila! Kibana is now connected to your Elasticsearch data. Kibana displays a read-only list of fields configured for the matching index.

## start exploring your data!

You’re ready to dive in to your data:

* Search and browse your data interactively from the [Discover](./discover.md) page.
* Chart and map your data from the [Visualize](./visualize.md) page.
* Create and view custom dashboards from the [Dashboard](./dashboard.md) page.
