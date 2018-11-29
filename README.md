# Description

This role configures an [ElasticSearch](https://www.elastic.co/guide/en/elasticsearch/reference/6.3/index.html) cluster as part of the [ELK Stack](https://www.elastic.co/elk-stack) for the purpose of storing logs for future querying. This data is aggregated by [Logstash](../logstash) for use with the [Kibana](../kibana) dashboard.

# Usage

The recommended number of hosts in an ES cluster is at least 3. This way with `number_of_replicas` set to `2` means that if one host goes down we lose none of the data.

For more details read:
https://www.elastic.co/guide/en/elasticsearch/guide/current/replica-shards.html

# Configuration

The only mandatory configuration is the cluster name in [`defaults/main.yml`](defaults/main.yml):
```yaml
es_cluster_name: 'my-awesome-cluster'
```

The only other configuration that makes any difference are the JVM options like the ones related to heap size in:
```yaml
es_jvm_min_heap: 2g
es_jvm_max_heap: 2g
```

As the hosts are scaled up to deal with more and more logs we should adjust those in turn.

# Known Issues

Because we need to know the Tinc VPN IPs of all the nodes in the ES cluster we need to run the `setup` modules(`gather_facts: true`) on them in order to get that. So if this role is not ran for the whole cluster it will fail due to lack of value `ansible_local.tinc.vpn_ip` variable.

We could use Consul for this but it would not work the first time setting up a new cluster.
