# Description

This document gives and overview of how creating backups in ElasticSearch works.

# Usage

We mount a `node/backup` folder in the container under `/var/backup`.

This folder can be used to create snapshots, for example:
```sh
 > PAYLOAD='{"type":"fs","settings":{"location":"/var/backup/kibana"}}'
 > curl -sSf -XPUT localhost:9200/_snapshot/kibana -d "$PAYLOAD" -H 'content-type: application/json'
{"acknowledged":true}
```
Then you can create snapshots of all or individual indices:
```sh
 > PAYLOAD='{"indices":".kibana_*"}'
 > curl -sSf -XPUT localhost:9200/_snapshot/kibana/$(date +%Y%m%d) -d $PAYLOAD -H 'content-type: application/json'
{"accepted":true}
```
These snapshots can be looked up easily:
```sh
 > curl -sSf localhost:9200/_snapshot/kibana/20220214 | jq '.snapshots[] | { snapshot, indices }'
{
  "snapshot": "20220214",
  "indices": [
    ".kibana_8",
    ".kibana_7.14.2_001",
    ".kibana_2",
    ".kibana_6",
    ".kibana_task_manager_7.14.2_001",
    ".kibana_task_manager_1",
    ".kibana_9",
    ".kibana_7",
    ".kibana_3",
    ".kibana_1",
    ".kibana_4",
    ".kibana_5",
  ]
}
```
This snapshot can be restored just as easily:
```sh
 > curl -sSf -XPOST localhost:9200/_snapshot/kibana/20220214/_restore
{"accepted":true}
```
All these backups are available in the folder for backup with other tools:
```
 > du -hsc /docker/elasticsearch/node/backup/kibana/indices/*
400K	/docker/elasticsearch/node/backup/kibana/indices/5c9BpVRQRHGSCPcM-Xdndw
40K	/docker/elasticsearch/node/backup/kibana/indices/EGpsTrm1S5y6S7AvgmsZGg
38M	/docker/elasticsearch/node/backup/kibana/indices/H-AdzeWWQnWzp770CVHqjQ
1.1M	/docker/elasticsearch/node/backup/kibana/indices/MSKseBbUTwK46kNVjzKsfg
16K	/docker/elasticsearch/node/backup/kibana/indices/V1HHFmBHQu66lEP-acWwiw
2.8M	/docker/elasticsearch/node/backup/kibana/indices/VYuYnxYBS12wzbCuO7RiLw
612K	/docker/elasticsearch/node/backup/kibana/indices/fk1ZZamBSUS0tc-S-BZAHA
43M	total
```

# Links

* https://www.elastic.co/guide/en/elasticsearch/reference/current/searchable-snapshots.html
* https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html
* https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshot-restore.html
* https://discuss.elastic.co/t/location-doesnt-match-any-of-the-locations-specified-by-path-repo-because-this-setting-is-empty/38579/3
