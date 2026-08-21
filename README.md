# Description

This role configures an [ElasticSearch](https://www.elastic.co/guide/en/elasticsearch/reference/6.3/index.html) cluster as part of the [ELK Stack](https://www.elastic.co/elk-stack) for the purpose of storing logs for future querying. This data is aggregated by [Logstash](../logstash) for use with the [Kibana](../kibana) dashboard.

# Usage

The recommended number of hosts in an ES cluster is at least 3. This way with `number_of_replicas` set to `2` means that if one host goes down we lose none of the data.

For more details read:
https://www.elastic.co/guide/en/elasticsearch/guide/current/replica-shards.html

# Configuration

The only mandatory settings in [`defaults/main.yml`](defaults/main.yml) are:
```yaml
es_cluster_name: 'my-awesome-cluster'
es_master_nodes:
  - { name: node-01.es.example.vpn, addr: 1.2.3.4, port: 9300 }
  - { name: node-02.es.example.vpn, addr: 2.3.4.5, port: 9300 }
  - { name: node-03.es.example.vpn, addr: 3.4.5.6, port: 9300 }
```

The only other configuration that makes any difference are the JVM options like the ones related to heap size in:
```yaml
es_jvm_min_heap: 2g
es_jvm_max_heap: 2g
```

As the hosts are scaled up to deal with more and more logs we should adjust those in turn.

# Authentication

When Security module is enabled peer transport SSL certificates are required:
```yaml
es_security_enabled: true
es_transport_certs_path: '{{ es_node_host_config_path }}/certs'
es_transport_pem_path: '{{ es_transport_certs_path }}/{{ hostname }}.wg.pem'
es_transport_key_path: '{{ es_transport_certs_path }}/{{ hostname }}.wg.key'
es_transport_ca_pem_path: '{{ es_transport_certs_path }}/{{ vault_pki_ca_key_name }}.pem'
# Built-in admin user
es_admin_username: 'elastic' # HARDCODED
es_admin_password: '{{ lookup("vault", "elasticsearch/users", field="elastic") }}'
```
If additional roles and users are necessary, or changing passwords of built-in users, use:
```yaml
es_roles:
  - name: 'logstash_writer'
    cluster: ['monitor', 'manage_index_templates']
    indices:
      - names: ['logstash-*']
        privileges: ['write', 'create', 'create_index']
es_users:
  - name: 'logstash'
    pass: 'super_secret_password'
    full_name: 'Logstash Writer User'
    roles: ['logstash_writer']
    enabled: true

  # Built-in users
  - name: 'apm_system'
    pass: 'definitely_not_apm_system_pass'
  - name: 'kibana_system'
    pass: 'definitely_not_kibana_system_pass'
```

# Backups

For information on how to create backups see the [`BACKUPS.md`](./BACKUPS.md) document.

# Known Issues

Because we need to know the VPN IPs of all the nodes in the ES cluster we need to run the `setup` modules(`gather_facts: true`) on them in order to get that. So if this role is not ran for the whole cluster it will fail due to lack of value `ansible_local.wireguard.vpn_ip` variable.

We could use Consul for this but it would not work the first time setting up a new cluster.
