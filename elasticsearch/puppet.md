# Puppet 自动部署 Elasticsearch

<https://github.com/elastic/puppet-elasticsearch>

```
class { 'elasticsearch':
  version => '1.5.2',
  config => { 'cluster.name' => 'es1003' },
  java_install => true,
}
elasticsearch::instance { $fqdn:
  config => { 'node.name' => $fqdn },
  init_defaults => { 'ES_USER' => 'elasticsearch', 'ES_HEAP_SIZE' => $memorysize > 64 ? '31g' : $memorysize / 2 },
  datadir => [ '/data1/elasticsearch' ],
}
elasticsearch::template { 'templatename':
  host => $::ipaddress,
  port => 9200,
  content => '{"template":"*","settings":{"number_of_replicas":0}}'
}
