---
title: Configure the shared services
description: Size and connect the search engine, Redis, RabbitMQ, S3 storage, PostgreSQL and mail server that the XTM modules share, or point them to your own.
doc_type: how-to
products:
  - opencti
  - openaev
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/configuration/
  - openaev:/deployment/configuration/
---

This page covers the services the XTM modules share: the search engine, Redis, RabbitMQ, S3 object storage, PostgreSQL and the mail server. For each one it says which module uses it, what xtm-docker sets, the settings you decide on to replace the bundled container with your own service, such as a managed cluster, to secure and to size it, and what to watch. Every other key is in the generated references: [OpenCTI configuration](/reference/opencti/configuration/) and [OpenAEV configuration](/reference/openaev/configuration/).

## Before you start

- The stack installed as [Install with Docker Compose](/deploy/install/docker-compose/) describes. The settings below go in `.env` or in the `environment` of a service in `docker-compose.override.yml`: see [Change the stack without editing docker-compose.yml](/deploy/install/docker-compose/#change-the-stack-without-editing-docker-compose-yml).
- How each platform names its settings in the environment:
  - OpenCTI: two underscores per level of `default.json`. `elasticsearch:url` becomes `ELASTICSEARCH__URL`.
  - OpenAEV: one underscore per dot of the Spring property, hyphens kept. `openaev.rabbitmq.queue-type` becomes `OPENAEV_RABBITMQ_QUEUE-TYPE`.
  - XTM One: the variable names of its own configuration, such as `REDIS_URL`.

## Who uses which service

::diagram{id=deploy-configure-shared-services}

| Service | Service in xtm-docker | Image | Used by | Compose file |
| --- | --- | --- | --- | --- |
| Elasticsearch | `elasticsearch` | `elasticsearch:8.19.23` | OpenCTI, OpenAEV | `infra/elasticsearch.yml` |
| RabbitMQ | `rabbitmq` | `rabbitmq:4.3-management` | OpenCTI, OpenAEV | `infra/rabbitmq.yml` |
| Redis | `redis` | `redis:8.8.3` | OpenCTI, XTM One | `infra/redis.yml` |
| S3 object storage | `minio` | `rustfs/rustfs:1.0.4` (RustFS) | OpenCTI, OpenAEV, OpenCRQ, XTM One | `infra/rustfs.yml` |
| PostgreSQL with pgvector | `pgsql` | `pgvector/pgvector:pg17` | OpenAEV, OpenCRQ, XTM One | `infra/postgresql.yml` |

Every shared service starts with the whole stack, as `docker-compose.yml` includes the five Compose files of `infra/`.

Each module keeps its data apart in the shared services: its own index prefix in Elasticsearch, its own bucket in S3, its own database in PostgreSQL. You can therefore replace one shared service by your own for every module at once, or for one module only.

To point a module to your own service, set its variables below on the module's service in `docker-compose.override.yml`, and remove the bundled service from the `depends_on` list of that module with `!reset null`. When no module depends on the bundled service any more, give it a profile you never turn on, such as `disabled`, or Docker Compose keeps starting it, then remove its container with `docker compose rm -sf <service>`. For example, for an Elasticsearch cluster of your own used by OpenCTI and OpenAEV:

```yaml
services:
  opencti:
    environment:
      - ELASTICSEARCH__URL=https://search.example.com:9200
    depends_on:
      elasticsearch: !reset null
  openaev:
    environment:
      - ENGINE_URL=https://search.example.com:9200
      - SPRING_ELASTICSEARCH_URIS=https://search.example.com:9200
    depends_on:
      elasticsearch: !reset null
  elasticsearch:
    profiles: !override ["disabled"]
```

## Elasticsearch or OpenSearch

The bundled `elasticsearch` service is one node, with security turned off (`xpack.security.enabled=false`): no user, no TLS. Keep it for one host on a private network; in production, use a cluster with authentication.

:::steps
1. Size the Java heap. `ELASTIC_MEMORY_SIZE` in `.env` sets both `-Xms` and `-Xmx` of the bundled node.

   | RAM of the host | `ELASTIC_MEMORY_SIZE` |
   | --- | --- |
   | 16 GB | `2G` |
   | 32 GB | `4G` |
   | 64 GB or more | `8G` |

2. On a Linux host, raise the number of memory maps Elasticsearch may use, or it can fail to start.

   ```bash
   sudo sysctl -w vm.max_map_count=262144
   ```

   To keep the value after a reboot, add `vm.max_map_count=262144` to `/etc/sysctl.conf`.

3. To use your own cluster, set its address and credentials on each platform.

   :::::tabs
   ::::tab[OpenCTI]
   | Variable | Default | Set it to |
   | --- | --- | --- |
   | `ELASTICSEARCH__URL` | `http://localhost:9200` | The URL of your cluster, or a list of URLs. xtm-docker sets `http://elasticsearch:9200` |
   | `ELASTICSEARCH__USERNAME`, `ELASTICSEARCH__PASSWORD` | | The user of OpenCTI on the cluster |
   | `ELASTICSEARCH__API_KEY` | | An Elasticsearch API key instead of a user: the `encoded` value Elasticsearch returns when it creates the key |
   | `ELASTICSEARCH__INDEX_PREFIX` | `opencti` | Keep it, unless two OpenCTI platforms share the cluster |
   | `ELASTICSEARCH__NUMBER_OF_REPLICAS` | | xtm-docker sets `0`, for one node. On a cluster, set the number of replicas each index needs |
   | `ELASTICSEARCH__PROXY` | | The URL of an HTTP proxy between OpenCTI and the cluster, such as `http://proxy.example.com:3128`. The platforms' `HTTPS_PROXY` settings, in [Trust your certificate authority and proxy](/deploy/configure/certificates/), do not apply to the cluster |
   | `OPENSEARCH__REGION` | | The AWS region of an Amazon OpenSearch Service domain, such as `eu-west-3`. Requests are then signed with AWS credentials |
   | `OPENSEARCH__SERVICE` | `es` | With `OPENSEARCH__REGION`: `es` for Amazon OpenSearch Service, `aoss` for Amazon OpenSearch Serverless |

   :::only{opencti=">=7.261109"}
   OpenCTI chooses its client at start. With `ELASTICSEARCH__ENGINE_CLIENT=auto`, the default, it reads the answer of the cluster to `GET /`, with the URL, credentials, CA and proxy above: OpenSearch gets the OpenSearch client, Elasticsearch 9 and Elastic Serverless the Elasticsearch 9 client, any other version the Elasticsearch 8 client. With `OPENSEARCH__REGION`, it takes the OpenSearch client without asking. When the user of OpenCTI may not read `GET /`, or a proxy blocks it, force the client with `ELASTICSEARCH__ENGINE_CLIENT`: `elasticsearch8`, `elasticsearch9` or `opensearch` (`opensearch` only, with `OPENSEARCH__REGION`). OpenCTI does not start when the value is another one, or when the cluster does not answer.

   OpenCTI logs the client it selected, such as `Search engine client elasticsearch9 selected, detected from elk 9.1.0`. It warns that Elasticsearch versions before 9 are deprecated when it detects one. `ELASTICSEARCH__ENGINE_SELECTOR` and `ELASTICSEARCH__ENGINE_CHECK` no longer apply: OpenCTI ignores them and logs a warning while they are set; remove them.
   :::

   :::only{opencti="<7.261109"}
   | Variable | Default | Set it to |
   | --- | --- | --- |
   | `ELASTICSEARCH__ENGINE_SELECTOR` | `auto` | `elk` for Elasticsearch or `opensearch` for OpenSearch. `auto` detects the engine. Set `elk` with `ELASTICSEARCH__API_KEY`, so that OpenCTI uses the Elasticsearch client from the start |
   | `ELASTICSEARCH__ENGINE_CHECK` | `true` | With `elk` or `opensearch` selected, OpenCTI refuses to start when the engine it detects is the other one. `false` skips that check |
   :::
   ::::
   ::::tab[OpenAEV]
   | Variable | Default | Set it to |
   | --- | --- | --- |
   | `ENGINE_URL` | `http://localhost:9200` | The URL of your cluster. xtm-docker sets `http://elasticsearch:9200`, and the same value in `SPRING_ELASTICSEARCH_URIS` |
   | `ENGINE_USERNAME`, `ENGINE_PASSWORD` | | The user of OpenAEV on the cluster |
   | `ENGINE_CLIENT` | `auto` | `elasticsearch8`, `elasticsearch9` or `opensearch` to force the client. `auto` detects the engine from `ENGINE_URL` |
   | `ENGINE_ENGINE_AWS_MODE` | `no` | `es` for Amazon OpenSearch Service or `aoss` for Amazon OpenSearch Serverless, with `ENGINE_ENGINE_AWS_HOST` (the host, without `https://`) and `ENGINE_ENGINE_AWS_REGION` |

   OpenAEV indices start with `openaev`, so OpenCTI and OpenAEV can share one cluster.

   With `ENGINE_ENGINE_AWS_MODE`, OpenAEV signs its requests with the AWS credentials of its environment, such as the IAM role of the node or the pod it runs on. In the security settings of OpenSearch, map that IAM role as a backend role of a role allowed to manage the `openaev*` indices.

   OpenAEV rebuilds its indices from PostgreSQL by itself: pointed to a new, empty cluster, it creates each index at start and fills it again, so nothing needs to be copied. To rebuild the indices of the current cluster, stop OpenAEV, empty the `indexing_status` table of the `openaev` database, and start it again. The search screens fill up again while it reindexes. `engine.engine-selector`, which older versions used to choose the engine, is ignored now: set `ENGINE_CLIENT` instead.
   ::::
   :::::

   To trust the certificate of your cluster, see [Trust your certificate authority and proxy](/deploy/configure/certificates/).
:::

### Size how OpenCTI uses the cluster

The defaults fit most platforms. Change them when the logs of OpenCTI show search engine timeouts or rejected requests, or when indexing fails on a limit. Index settings (shards, replicas, result window) go into the index templates of OpenCTI at each start: they apply to the indices created afterwards, at the next [rollover](/deploy/operate/rollover/), not to the existing ones.

| Variable | Default | Effect |
| --- | --- | --- |
| `ELASTICSEARCH__NUMBER_OF_SHARDS` | The engine's default | Primary shards of each new index. Rollover keeps each primary shard under 50 GB: see [Manage Elasticsearch indices](/deploy/operate/rollover/) |
| `ELASTICSEARCH__REQUEST_TIMEOUT` | `3600000` (ms, one hour) | Time OpenCTI waits for one request to the cluster |
| `ELASTICSEARCH__MAX_RETRIES` | `3` | Retries of a failed request before OpenCTI returns an error |
| `ELASTICSEARCH__SNIFF_ON_START` | `false` | `true` to discover the nodes of the cluster at start and spread requests over them. Keep `false` behind a load balancer, or when the nodes announce addresses OpenCTI cannot reach |
| `ELASTICSEARCH__MAX_BULK_OPERATIONS` | `5000` | Operations in one bulk request during mass updates and migrations. Lower it if bulk requests time out or exceed the request size limit of the cluster |
| `ELASTICSEARCH__MAX_CONCURRENCY` | `4` | Bulk requests sent in parallel during mass updates and migrations. Raise it on a large cluster to go faster, lower it to reduce the load |
| `ELASTICSEARCH__BATCH_LOADER_MAX_SIZE` | `300` | Elements grouped into one query when an API request loads related data, such as creators and markings |
| `ELASTICSEARCH__DEFAULT_PAGINATION_RESULT`, `ELASTICSEARCH__MAX_PAGINATION_RESULT` | `500`, `5000` | Page size of a list query that asks for none, and the largest page size a query gets |
| `ELASTICSEARCH__MAX_RESULT_WINDOW` | `100000` | The `max_result_window` of the indices: the deepest offset a search can reach |
| `ELASTICSEARCH__MAX_RUNTIME_RESOLUTIONS` | `5000` | Entities loaded to sort a list by a computed column, such as the author. Raise it if such a sort shows `Unknown` on a large dataset |
| `ELASTICSEARCH__MAX_CONTAINER_RESOLUTIONS`, `ELASTICSEARCH__MAX_CONTAINER_OBJECT_RESOLUTIONS` | `1000`, `100000` | Containers, and objects in them, loaded to show the related containers of an entity |
| `ELASTICSEARCH__UNSECURE_SCRIPT_FILTER_ENABLED` | `false` | `true` allows the `script` filter operator, which sends a raw script to the cluster. Any user who can filter can then run a script: keep `false` unless you trust every user |

:::only{opencti=">=7.261030"}
`ELASTICSEARCH__MAX_MAPPING_FIELDS`, `3500` by default, sets `index.mapping.total_fields.limit` on the templates and on the existing indices at each start. Raise it if indexing fails because an index reached its field limit, which many custom attributes can cause.
:::

What to watch on the cluster: the heap of each node, the rejected requests of its `write` and `search` thread pools, and the size of the shards of the `opencti_*` indices.

All settings: [Elasticsearch](/reference/opencti/configuration/#elasticsearch).

## Redis

OpenCTI keeps its event stream in Redis, among other things; XTM One uses it too. The bundled `redis` service has no password.

::::tabs
:::tab[OpenCTI]
| Variable | Default | Set it to |
| --- | --- | --- |
| `REDIS__HOSTNAME`, `REDIS__PORT` | `localhost`, `6379` | Your Redis. xtm-docker sets `redis` and `6379` |
| `REDIS__USERNAME`, `REDIS__PASSWORD` | | The credentials of your Redis |
| `REDIS__MODE` | `single` | `sentinel` or `cluster`, with `REDIS__HOSTNAMES` (and `REDIS__SENTINEL_MASTER_NAME` in sentinel mode) |
| `REDIS__USE_SSL` | `false` | `true` for a Redis that only accepts TLS, in single or cluster mode |
| `REDIS__DATABASE` | `0` | The database number, in single mode only |
| `REDIS__NAMESPACE` | | A prefix for every key and channel of OpenCTI, such as `cti-prod`. Set it when several OpenCTI platforms share one Redis |
| `REDIS__HOST_IP_FAMILY` | `4` | `6` for a Redis reachable over IPv6 only, `0` for both |
| `REDIS__TRIMMING` | `2000000` | The number of events kept in the live stream. It sizes the memory Redis needs |
| `REDIS__ACTIVITY_TRIMMING`, `REDIS__NOTIFICATION_TRIMMING` | `50000`, `50000` | The events kept, approximately, in the activity stream and in the notification stream. Raise the second if digests over long periods miss events |
:::
:::tab[XTM One]
| Variable | Default | Set it to |
| --- | --- | --- |
| `REDIS_URL` | `redis://redis:6379` | The URL of your Redis. Set it on `xtm-one` and `xtm-one-worker` |
| `REDIS_PASSWORD` | | The password of your Redis |
| `REDIS_MODE` | `standalone` | `sentinel`, with `REDIS_SENTINEL_HOSTS` and `REDIS_SENTINEL_SERVICE` |
:::
::::

### OpenCTI with Redis Sentinel

In sentinel mode, OpenCTI asks the Sentinels listed in `REDIS__HOSTNAMES` for the address of the master named `REDIS__SENTINEL_MASTER_NAME`, then connects to that master.

| Variable | Default | Set it to |
| --- | --- | --- |
| `REDIS__SENTINEL_USERNAME`, `REDIS__SENTINEL_PASSWORD` | | The credentials of the Sentinels, when they ask for some. `REDIS__USERNAME` and `REDIS__PASSWORD` stay those of the master |
| `REDIS__SENTINEL_ROLE` | `master` | Keep `master`: OpenCTI writes to Redis. `slave` connects to a replica, in the order of `REDIS__SENTINEL_PREFERRED_SLAVES`, a list of `{ "ip", "port", "prio" }` objects |
| `REDIS__SENTINEL_FAILOVER_DETECTOR` | `false` | `true` to listen to the Sentinels and reconnect to the new master as soon as a failover happens, instead of when the connection to the old one fails |
| `REDIS__SENTINEL_UPDATE_SENTINELS` | `true` | `false` to use only the Sentinels of `REDIS__HOSTNAMES`, when the addresses the Sentinels announce are not reachable from OpenCTI |
| `REDIS__SENTINEL_TLS` | `false` | See the warning below |

:::warning
In sentinel mode, OpenCTI passes no TLS settings to its Redis client: `REDIS__USE_SSL`, `REDIS__CA` and `REDIS__SENTINEL_TLS` do not encrypt the connections to the Sentinels or to the master. Keep them on a private network, or use single or cluster mode for TLS.
:::

```yaml
services:
  opencti:
    environment:
      - REDIS__MODE=sentinel
      - 'REDIS__HOSTNAMES=["sentinel1.example.com:26379","sentinel2.example.com:26379","sentinel3.example.com:26379"]'
      - REDIS__SENTINEL_MASTER_NAME=opencti
      - REDIS__SENTINEL_USERNAME=opencti
      - REDIS__SENTINEL_PASSWORD=${REDIS_SENTINEL_PASSWORD}
      - REDIS__USERNAME=opencti
      - REDIS__PASSWORD=${REDIS_PASSWORD}
      - REDIS__SENTINEL_FAILOVER_DETECTOR=true
```

### OpenCTI with a Redis cluster

In cluster mode, OpenCTI connects to the nodes of `REDIS__HOSTNAMES` and learns the others from the cluster. Putting Redis in cluster mode for capacity is in [Scale](/deploy/operate/scale/#redis).

| Variable | Default | Set it to |
| --- | --- | --- |
| `REDIS__NAT_MAP` | | When the nodes sit behind NAT or a proxy (Docker, Kubernetes, cloud) and announce addresses OpenCTI cannot reach: a JSON list of `announced>reachable` pairs, such as `["10.0.0.11:6379>redis-1.example.com:6379"]` |
| `REDIS__SCALE_READS` | `all` | The nodes that serve reads: `master`, `slave` or `all`. `master` when a read must never miss a write a replica has not received yet |

:::only{opencti=">=7.261030"}
With TLS (`REDIS__USE_SSL=true`), OpenCTI checks the certificate of every node against `REDIS__TLS_SERVERNAME`, or `REDIS__HOSTNAME` when it is empty, and sends that name in the TLS handshake (SNI). In single mode the same name applies to the one server. Set `REDIS__TLS_CLUSTER_NODE_MODE=true` to check the certificate of each node against that node's own address instead.
:::

:::only{opencti=">=7.260811"}
`REDIS__NOTIFICATION_RANGE_BATCH_SIZE`, `1000` by default, sets how many events of the notification stream the notification manager reads per request when it builds a digest. Lower it if building digests over a busy stream takes too much memory.
:::

:::only{opencti=">=7.261030"}
`REDIS__MAX_EVENT_LENGTH`, in bytes, moves an event larger than this size out of the live stream into S3 storage, and keeps only its file name in Redis. `0`, the default, keeps every event in Redis. Set it, such as `1048576` for 1 MB, when large bundles make Redis use too much memory.
:::

What to watch on Redis: its used memory against `maxmemory`, which the stream trimming settings above drive, and its connected clients. Each OpenCTI node opens six connections at start (five, and one for its lock process), then one more for each reader of the stream it runs: each manager that reads the stream (history, rules, notifications, playbooks, publication, activity, file indexing) and each live stream a connector or a remote platform consumes from it.

All settings: [Redis](/reference/opencti/configuration/#redis).

## RabbitMQ

OpenCTI and OpenAEV pass work to their workers and integrations through RabbitMQ. In xtm-docker both use the same broker and the same user, `RABBITMQ_DEFAULT_USER` and `RABBITMQ_DEFAULT_PASS` from `.env`. OpenCTI also calls the management API, on port 15672, and so does OpenAEV when it migrates its queues: a broker of your own needs the management plugin.

::::tabs
:::tab[OpenCTI]
| Variable | Default | Set it to |
| --- | --- | --- |
| `RABBITMQ__HOSTNAME`, `RABBITMQ__PORT` | `localhost`, `5672` | Your broker |
| `RABBITMQ__PORT_MANAGEMENT` | `15672` | The port of its management API. `RABBITMQ__HOSTNAME_MANAGEMENT` if it has another host name |
| `RABBITMQ__USERNAME`, `RABBITMQ__PASSWORD` | `guest`, `guest` | The user of OpenCTI |
| `RABBITMQ__VHOST` | `/` | The virtual host of OpenCTI |
| `RABBITMQ__QUEUE_TYPE` | `classic` | `quorum` on a cluster of brokers |
| `RABBITMQ__QUEUE_PREFIX` | | A prefix, followed by `_`, on the names of the exchanges and queues of OpenCTI, such as `cti-prod`. Set it, or a virtual host of its own, when several OpenCTI platforms share one broker |
| `RABBITMQ__USE_SSL`, `RABBITMQ__MANAGEMENT_SSL` | `false`, `false` | `true` for a broker, or a management API, that only accepts TLS |
| `RABBITMQ__USE_SSL_REJECT_UNAUTHORIZED`, `RABBITMQ__MANAGEMENT_SSL_REJECT_UNAUTHORIZED` | off, off | `true` to check the certificate of the broker, or of the management API. Off by default: turn them on once OpenCTI trusts their CA, see [Trust your certificate authority and proxy](/deploy/configure/certificates/) |
| `RABBITMQ__USE_SSL_CERT`, `RABBITMQ__USE_SSL_KEY` | | For a broker that asks for a client certificate: the certificate and the private key of OpenCTI, as paths in the container or PEM contents. Or `RABBITMQ__USE_SSL_PFX`, both in one PKCS#12 file |
| `RABBITMQ__USE_SSL_PASSPHRASE` | | The passphrase of that private key or PKCS#12 file, when it is encrypted |
:::
:::tab[OpenAEV]
| Variable | Default | Set it to |
| --- | --- | --- |
| `OPENAEV_RABBITMQ_HOSTNAME`, `OPENAEV_RABBITMQ_PORT` | `localhost`, `5672` | Your broker |
| `OPENAEV_RABBITMQ_MANAGEMENT-PORT` | `15672` | The port of its management API |
| `OPENAEV_RABBITMQ_USER`, `OPENAEV_RABBITMQ_PASS` | `guest`, `guest` | The user of OpenAEV |
| `OPENAEV_RABBITMQ_VHOST` | `/` | The virtual host of OpenAEV |
| `OPENAEV_RABBITMQ_QUEUE-TYPE` | `classic` | `quorum` on a cluster of brokers |
| `OPENAEV_RABBITMQ_PREFIX` | `openaev` | The prefix of the OpenAEV queues |
| `OPENAEV_RABBITMQ_SSL` | `false` | `true` for a broker that only accepts TLS. OpenAEV then also calls the management API over HTTPS, on the host of the broker |
| `OPENAEV_RABBITMQ_TRUST-STORE`, `OPENAEV_RABBITMQ_TRUST-STORE-PASSWORD` | | Over TLS, a PKCS12 trust store that holds the CA of the broker's certificate, such as `file:/opt/openaev/rabbitmq-truststore.p12`, and its password. OpenAEV then checks the broker against this trust store only, in TLS 1.3, and the management API against this trust store or the platform's. Without it, or when the file does not exist, as with the placeholder of the shipped `application.properties`, OpenAEV checks both against the platform trust store: the CAs of Java and the certificates of `OPENAEV_EXTRA-TRUSTED-CERTS-DIR` |

Over TLS, OpenAEV always checks the certificate of the broker and of its management API: `OPENAEV_RABBITMQ_MANAGEMENT-INSECURE` no longer has any effect. When your internal CA signed the certificate of the broker, the simplest is to put that CA in the folder of `OPENAEV_EXTRA-TRUSTED-CERTS-DIR`, as [Trust your certificate authority and proxy](/deploy/configure/certificates/) shows, and to set no RabbitMQ trust store: the broker and the management API then trust the same CAs as the other connections of OpenAEV. For a broker that asks for a client certificate, OpenAEV presents the one of the key store that the standard Java properties `javax.net.ssl.keyStore`, `javax.net.ssl.keyStoreType` and `javax.net.ssl.keyStorePassword` name.

For example, for a broker whose certificate the internal CA of `./certs/internal-ca.pem` signed, in `docker-compose.override.yml`:

```yaml
services:
  openaev:
    volumes:
      - ./certs:/opt/openaev/additional_certs:ro
    environment:
      - OPENAEV_RABBITMQ_HOSTNAME=rabbitmq.example.com
      - OPENAEV_RABBITMQ_PORT=5671
      - OPENAEV_RABBITMQ_MANAGEMENT-PORT=15671
      - OPENAEV_RABBITMQ_USER=openaev
      - OPENAEV_RABBITMQ_PASS=${OPENAEV_RABBITMQ_PASSWORD}
      - OPENAEV_RABBITMQ_SSL=true
    depends_on:
      rabbitmq: !reset null
```
:::
::::

:::warning
Integrations connect to RabbitMQ with the host name and port their platform gives them, `rabbitmq` and `5672` in xtm-docker. An integration that runs where that name does not resolve cannot reach its queue. See [Run an integration without XTM Composer](/deploy/integrations/manual/).
:::

OpenCTI workers and connectors get the host, port, user and `RABBITMQ__USE_SSL` setting of the broker from OpenCTI, not its certificates. Over TLS, they accept any certificate the broker presents until you set, on each worker and connector:

| Variable | Default | Set it to |
| --- | --- | --- |
| `MQ_USE_SSL_REJECT_UNAUTHORIZED` | `false` | `true` to check the certificate of the broker |
| `MQ_USE_SSL_CA` | | The CA that signed it, as a path in the container or PEM content |
| `MQ_USE_SSL_CERT`, `MQ_USE_SSL_KEY` | | For a broker that asks for a client certificate: the certificate and its private key, as paths or PEM content |
| `MQ_USE_SSL_PASSPHRASE` | | The passphrase of that private key, when it is encrypted |

:::only{opencti=">=7.261119"}
When `MQ_USE_SSL_REJECT_UNAUTHORIZED` is unset, its default is the value of `OPENCTI_TLS_REJECT_UNAUTHORIZED`, `false` when that one is unset too: `OPENCTI_TLS_REJECT_UNAUTHORIZED=true` on a worker or connector also turns this check on.
:::

What to watch on RabbitMQ: the messages ready in the `push_` queues, which grow when the workers cannot keep up (add workers, see [Scale](/deploy/operate/scale/#add-workers)), the disk and memory alarms of the broker, which block every publisher, and its free disk space.

All settings of OpenCTI: [RabbitMQ](/reference/opencti/configuration/#rabbitmq).

## S3 object storage

OpenCTI, OpenAEV, OpenCRQ and XTM One store their files in S3. The bundled `minio` service runs RustFS, an S3-compatible server, with the credentials `MINIO_ROOT_USER` and `MINIO_ROOT_PASSWORD` of `.env`: the service and the variables keep the names of MinIO, which the stack ran before. Each platform uses its own bucket.

::::tabs
:::tab[OpenCTI]
| Variable | Default | Set it to |
| --- | --- | --- |
| `MINIO__ENDPOINT`, `MINIO__PORT` | `localhost`, `9000` | Your S3 service, such as `s3.eu-west-3.amazonaws.com` and `443` for AWS. With `s3.amazonaws.com`, the AWS SDK chooses the endpoint of the region itself |
| `MINIO__USE_SSL` | `false` | `true` for HTTPS |
| `MINIO__ACCESS_KEY`, `MINIO__SECRET_KEY` | `ChangeMe`, `ChangeMe` | The credentials of OpenCTI. xtm-docker sets `MINIO_ROOT_USER` and `MINIO_ROOT_PASSWORD` of `.env` |
| `MINIO__SESSION_TOKEN` | | The session token of temporary credentials from AWS Security Token Service (STS), with the access and secret keys. OpenCTI reads it at start and does not renew it: restart it with new credentials before they expire, or prefer `MINIO__USE_AWS_ROLE` |
| `MINIO__BUCKET_NAME` | `opencti-bucket` | The bucket of OpenCTI |
| `MINIO__BUCKET_REGION` | `us-east-1` | The region of the bucket on AWS |
| `MINIO__USE_AWS_ROLE` | `false` | `true` to take the credentials from the AWS role of the host instead of the access and secret keys |
| `MINIO__DISABLE_CHECKSUM_VALIDATION` | `false` | `true` when an S3-compatible service rejects uploads because of the checksum headers recent AWS SDKs send. OpenCTI then sends checksums only when the operation requires them |
| `MINIO__USE_AWS_LOGS` | `false` | `true` to write the debug messages of the S3 client to the log of OpenCTI, to diagnose a connection to the bucket |
| `MINIO__EXCLUDED_FILES` | `[".DS_Store"]` | The file names OpenCTI hides when it lists the files of the bucket, case-insensitive |

All settings: [S3 Bucket](/reference/opencti/configuration/#s3-bucket).
:::
:::tab[OpenAEV]
| Variable | Default | Set it to |
| --- | --- | --- |
| `MINIO_ENDPOINT`, `MINIO_PORT` | `localhost`, `9000` | Your S3 service. On AWS, the endpoint of the bucket's region, such as `s3.eu-west-3.amazonaws.com`, and `443` |
| `MINIO_SECURE` | `false` | `true` for HTTPS |
| `MINIO_ACCESS-KEY`, `MINIO_ACCESS-SECRET` | | The credentials of OpenAEV |
| `MINIO_BUCKET` | `openaev` | The bucket of OpenAEV, created at start when missing |
| `OPENAEV_S3_USE-AWS-ROLE` | `false` | `true` to take the credentials from AWS Security Token Service (STS), with the IAM role of the host, instead of the access keys |
| `OPENAEV_S3_STS-ENDPOINT` | | Optional, experimental. Your own STS endpoint. Empty, OpenAEV calls `https://sts.amazonaws.com`, or the regional endpoint when `AWS_REGION` is set |
:::
:::tab[XTM One]
| Variable in `modules/xtm-one.yml` | Set from `.env` | Set it to |
| --- | --- | --- |
| `S3_ENDPOINT` | | Your S3 service, as `host:port`. xtm-docker sets `minio:9000` |
| `S3_USE_SSL` | | `true` for HTTPS |
| `S3_ACCESS_KEY`, `S3_SECRET_KEY` | `MINIO_ROOT_USER`, `MINIO_ROOT_PASSWORD` | The credentials of XTM One |
| `S3_BUCKET` | `XTM_ONE_S3_BUCKET`, `xtm-one-files` by default | The bucket of XTM One. XTM One creates it at its first start |
| `S3_REGION` | | The region of the bucket, required on AWS |
| `S3_USE_AWS_ROLE` | | `true` to take the credentials from the role of the host |

Set them on `xtm-one` and `xtm-one-worker`.
:::
:::tab[OpenCRQ]
| Variable in `modules/opencrq.yml` | Set from `.env` | Set it to |
| --- | --- | --- |
| `FILE_STORAGE__ENDPOINT` | | The URL of your S3 service. xtm-docker sets `http://minio:9000` |
| `FILE_STORAGE__ACCESS_KEY`, `FILE_STORAGE__SECRET_KEY` | `MINIO_ROOT_USER`, `MINIO_ROOT_PASSWORD` | The credentials of OpenCRQ |
| `FILE_STORAGE__BUCKET` | `OPENCRQ_S3_BUCKET`, `opencrq` by default | The bucket of OpenCRQ. OpenCRQ never creates it: in xtm-docker, the `opencrq-bucket-init` job does, at every start |
| `FILE_STORAGE__REGION` | | The region of the bucket, `eu-west-3` by default |

On your own S3 service, create the bucket first, and keep `opencrq-bucket-init` from starting, as the last step of [PostgreSQL](#postgresql) does for `opencrq-db-init`.
:::
::::

## PostgreSQL

OpenAEV, OpenCRQ and XTM One share the `pgsql` instance, each with its own database: `openaev`, `opencrq` and `xtm_one`.

- OpenAEV connects with the superuser of the instance, `POSTGRES_USER` and `POSTGRES_PASSWORD` of `.env`.
- XTM One connects with its own role, `XTM_ONE_POSTGRES_USER` and `XTM_ONE_POSTGRES_PASSWORD`, which must differ from the superuser. At every start, the `xtm-one-db-init` service creates that role, the `xtm_one` database and the `vector` extension (pgvector) that XTM One needs. Only a superuser can create that extension.
- OpenCRQ connects with its own role too, `OPENCRQ_POSTGRES_USER` (`opencrq`) and `OPENCRQ_POSTGRES_PASSWORD`, which must differ from the superuser and from the XTM One role. At every start, the `opencrq-db-init` service creates that role, sets its password from `.env`, and creates the `opencrq` database it owns. The only extension OpenCRQ creates, `unaccent`, is a trusted one: the role needs no superuser right. Set `OPENCRQ_POSTGRES_USER` once: changed after the first start, the `opencrq` database and its tables stay owned by the old role.
- `infra/postgresql.yml` starts PostgreSQL with `max_connections=200`: OpenAEV opens up to 20 connections, XTM One up to 35 per process, over its web processes and its worker, and OpenCRQ about 50 at peak. Keep at least this value on your own server.
- It also gives the container 1 GB of shared memory (`shm_size`): parallel queries pass their rows through it, and the aggregates of OpenCRQ fail past the 64 MB a container gets by default.

To use your own PostgreSQL server:

:::steps
1. On your server, connect with `psql` as a superuser and create the databases, the roles of XTM One and OpenCRQ, and the extension. Use the values of `XTM_ONE_POSTGRES_USER` (`xtmone` in `.env.sample`), `XTM_ONE_POSTGRES_PASSWORD`, `OPENCRQ_POSTGRES_USER` (`opencrq`) and `OPENCRQ_POSTGRES_PASSWORD` from `.env`.

   ```sql
   CREATE DATABASE openaev;
   CREATE ROLE opencrq LOGIN PASSWORD 'the value of OPENCRQ_POSTGRES_PASSWORD';
   CREATE DATABASE opencrq OWNER opencrq;
   CREATE ROLE xtmone LOGIN PASSWORD 'the value of XTM_ONE_POSTGRES_PASSWORD';
   CREATE DATABASE xtm_one OWNER xtmone;
   \connect xtm_one
   CREATE EXTENSION IF NOT EXISTS vector;
   ```

2. Point OpenAEV to it, in the `openaev` service of `docker-compose.override.yml`. Set `POSTGRES_USER` and `POSTGRES_PASSWORD` in `.env` to a role of your server that can create extensions in the `openaev` database, as the superuser does in xtm-docker: OpenAEV creates the `hstore` and `pg_trgm` extensions during its migrations.

   ```yaml
   - SPRING_DATASOURCE_URL=jdbc:postgresql://postgres.example.com:5432/openaev
   - SPRING_DATASOURCE_USERNAME=${POSTGRES_USER}
   - SPRING_DATASOURCE_PASSWORD=${POSTGRES_PASSWORD}
   ```

3. Point XTM One to it, in the `xtm-one` and `xtm-one-worker` services of `docker-compose.override.yml`:

   ```yaml
   - DATABASE_URL=postgresql+asyncpg://${XTM_ONE_POSTGRES_USER}:${XTM_ONE_POSTGRES_PASSWORD}@postgres.example.com:5432/xtm_one
   ```

4. Point OpenCRQ to it, in the `opencrq` service of `docker-compose.override.yml`. It keeps connecting as `OPENCRQ_POSTGRES_USER`:

   ```yaml
   - POSTGRES__HOST=postgres.example.com
   - POSTGRES__PORT=5432
   ```

5. Keep the `pgsql`, `xtm-one-db-init` and `opencrq-db-init` services from starting, and remove them from the `depends_on` lists of `openaev`, `xtm-one` and `opencrq`. With steps 2 to 4, `docker-compose.override.yml` holds:

   ```yaml
   services:
     openaev:
       environment:
         - SPRING_DATASOURCE_URL=jdbc:postgresql://postgres.example.com:5432/openaev
         - SPRING_DATASOURCE_USERNAME=${POSTGRES_USER}
         - SPRING_DATASOURCE_PASSWORD=${POSTGRES_PASSWORD}
       depends_on:
         pgsql: !reset null
     xtm-one:
       environment:
         - DATABASE_URL=postgresql+asyncpg://${XTM_ONE_POSTGRES_USER}:${XTM_ONE_POSTGRES_PASSWORD}@postgres.example.com:5432/xtm_one
       depends_on:
         xtm-one-db-init: !reset null
     xtm-one-worker:
       environment:
         - DATABASE_URL=postgresql+asyncpg://${XTM_ONE_POSTGRES_USER}:${XTM_ONE_POSTGRES_PASSWORD}@postgres.example.com:5432/xtm_one
     opencrq:
       environment:
         - POSTGRES__HOST=postgres.example.com
         - POSTGRES__PORT=5432
       depends_on:
         opencrq-db-init: !reset null
     pgsql:
       profiles: !override ["disabled"]
     xtm-one-db-init:
       profiles: !override ["disabled"]
     opencrq-db-init:
       profiles: !override ["disabled"]
   ```

   If they ran, remove their containers with `docker compose rm -sf pgsql xtm-one-db-init opencrq-db-init`.

   :::verify
   ```bash
   docker compose config --quiet && echo "configuration OK"
   ```
   It prints `configuration OK`. After `docker compose up -d`, `docker compose ps` shows `openaev`, `xtm-one` and `opencrq` healthy.
   :::
:::

## Mail server

The `SMTP_*` and `IMAP_*` variables of `.env` do not reach every platform:

| Platform | Reads |
| --- | --- |
| OpenAEV | Every `SMTP_*` variable, and the `IMAP_*` variables when `OPENAEV_MAIL_IMAP_ENABLED=true` |
| OpenCTI | `SMTP_HOST` only: `modules/opencti.yml` sets the port to 25, without authentication |

OpenAEV sends the email injects of simulations through SMTP. On port 587, as with Gmail or Microsoft 365, set `SMTP_STARTTLS_ENABLE=true` and `SMTP_SSL_ENABLE=false`; on port 465, the reverse. A mailbox protected by two-factor authentication needs an app password, created in the account of the mailbox, in `SMTP_PASSWORD`. With IMAP on, OpenAEV reads the same mailbox every 10 seconds, to show the players' replies in the simulation: it reads the folders of `OPENAEV_MAIL_IMAP_INBOX` (`INBOX`, several separated by commas) and files what it sends in `OPENAEV_MAIL_IMAP_SENT` (`Sent`), which it creates when missing. Gmail names that folder `[Gmail]/Sent Mail`. The keys are in the [spring.mail](/reference/openaev/configuration/#spring-mail) and [openaev.mail](/reference/openaev/configuration/#openaev-mail) blocks of the configuration reference.

To send OpenCTI mail through a relay that needs authentication, add these variables to the `opencti` service in `docker-compose.override.yml`. `SMTP__PORT` replaces the port 25 that `modules/opencti.yml` sets:

```yaml
services:
  opencti:
    environment:
      - SMTP__PORT=${SMTP_PORT}
      - SMTP__USERNAME=${SMTP_USERNAME}
      - SMTP__PASSWORD=${SMTP_PASSWORD}
      - SMTP__USE_SSL=${SMTP_SSL_ENABLE}
```

`SMTP__USE_SSL=true` opens the connection in TLS, as on port 465. On port 587, keep `SMTP_SSL_ENABLE=false`.

:::only{opencti=">=7.261127"}
On port 587, also add `SMTP__REQUIRE_TLS=true`: OpenCTI then sends its user name and password only after the server has upgraded the connection with STARTTLS, and sends nothing to a server that does not. Unset, `SMTP__REQUIRE_TLS` follows `TLS__REJECT_UNAUTHORIZED`, `false` by default, and OpenCTI warns at each start that `SMTP credentials may be sent over an unencrypted connection`, a warning that the `error` log level of xtm-docker hides. See [Check certificates on every outgoing connection of OpenCTI](/deploy/configure/certificates/#check-certificates-on-every-outgoing-connection-of-opencti).
:::

:::only{opencti=">=7.260811"}
For a provider that no longer accepts passwords, such as Microsoft 365, OpenCTI authenticates with OAuth2: set `SMTP__AUTH_TYPE=oauth2`, then `SMTP__OAUTH_USER` (the mailbox), `SMTP__OAUTH_CLIENT_ID`, `SMTP__OAUTH_CLIENT_SECRET`, `SMTP__OAUTH_ISSUER` (the OpenID Connect issuer URL of the provider) and `SMTP__OAUTH_REFRESH_TOKEN`, instead of `SMTP__USERNAME` and `SMTP__PASSWORD`. OpenCTI exchanges the refresh token for an access token before it sends. Request the refresh token with the scope that allows sending through SMTP: for Microsoft 365, `https://outlook.office.com/SMTP.Send` with `offline_access`; for Google Workspace, `https://mail.google.com/`.

OpenCTI can also take its mail server from its interface, in :menu[Settings > Security > SMTP configuration], without a restart. When that configuration is in use, OpenCTI ignores the `SMTP__*` variables. See [Send emails from OpenCTI](/opencti/administer/smtp/).
:::

:::only{opencti=">=7.261030"}
`SMTP__CONNECTION_TIMEOUT` and `SMTP__GREETING_TIMEOUT`, in milliseconds, apply in both cases: OpenCTI always reads them from its configuration.
:::

::::example[Point OpenCTI to the managed services of Example CERT]
Example CERT runs OpenCTI at https://cti.example.com for the hotel chains it serves. It moves OpenCTI to its own services: an Elasticsearch cluster of three nodes with an API key, a Redis Sentinel group, a RabbitMQ cluster behind a TLS load balancer, and an S3 bucket on AWS reached with the role of the host. Its internal CA signed the certificates of the cluster and of the broker; `./certs/internal-ca.pem` holds it. In `docker-compose.override.yml`:

```yaml
services:
  opencti:
    volumes:
      - ./certs:/opt/opencti/certs:ro
    environment:
      - ELASTICSEARCH__URL=https://search.example.com:9200
      - ELASTICSEARCH__API_KEY=${OPENCTI_ELASTIC_API_KEY}
      - ELASTICSEARCH__SSL__CA=/opt/opencti/certs/internal-ca.pem
      - ELASTICSEARCH__NUMBER_OF_SHARDS=3
      - ELASTICSEARCH__NUMBER_OF_REPLICAS=1
      - REDIS__MODE=sentinel
      - 'REDIS__HOSTNAMES=["sentinel1.example.com:26379","sentinel2.example.com:26379","sentinel3.example.com:26379"]'
      - REDIS__SENTINEL_MASTER_NAME=opencti
      - REDIS__SENTINEL_PASSWORD=${REDIS_SENTINEL_PASSWORD}
      - REDIS__PASSWORD=${REDIS_PASSWORD}
      - REDIS__SENTINEL_FAILOVER_DETECTOR=true
      - RABBITMQ__HOSTNAME=rabbitmq.example.com
      - RABBITMQ__PORT=5671
      - RABBITMQ__PORT_MANAGEMENT=15671
      - RABBITMQ__USERNAME=opencti
      - RABBITMQ__PASSWORD=${OPENCTI_RABBITMQ_PASSWORD}
      - RABBITMQ__QUEUE_TYPE=quorum
      - RABBITMQ__USE_SSL=true
      - 'RABBITMQ__USE_SSL_CA=["/opt/opencti/certs/internal-ca.pem"]'
      - RABBITMQ__USE_SSL_REJECT_UNAUTHORIZED=true
      - RABBITMQ__MANAGEMENT_SSL=true
      - RABBITMQ__MANAGEMENT_SSL_REJECT_UNAUTHORIZED=true
      - NODE_EXTRA_CA_CERTS=/opt/opencti/certs/internal-ca.pem
      - MINIO__ENDPOINT=s3.amazonaws.com
      - MINIO__PORT=443
      - MINIO__USE_SSL=true
      - MINIO__BUCKET_NAME=example-cert-opencti
      - MINIO__BUCKET_REGION=eu-west-3
      - MINIO__USE_AWS_ROLE=true
    depends_on:
      elasticsearch: !reset null
      redis: !reset null
      rabbitmq: !reset null
      minio: !reset null
```

:::only{opencti="<7.261109"}
With the API key, add `ELASTICSEARCH__ENGINE_SELECTOR=elk` to the `environment` list, so that OpenCTI uses the Elasticsearch client from the start.
:::

After `docker compose up -d opencti`, the query of [Check that it works](#check-that-it-works) lists `Search engine`, `RabbitMQ` and `Redis` with the versions of Example CERT's services. The bundled services still start for the other modules: give each one the `disabled` profile once no module uses it, as [Who uses which service](#who-uses-which-service) shows.
::::

::::only{opencti=">=7.261127"}

## Replace the default credentials of the services

At each start, OpenCTI checks that the credentials it uses for its shared services are not left at a known default value, ignoring case:

| Service | Settings checked | Reported when |
| --- | --- | --- |
| RabbitMQ | `RABBITMQ__USERNAME`, `RABBITMQ__PASSWORD` | The password is `guest` or `changeme`, as with the defaults of OpenCTI |
| S3 storage | `MINIO__ACCESS_KEY`, `MINIO__SECRET_KEY` | The secret key is `minioadmin` or `changeme`, as with the defaults of OpenCTI. Not checked with `MINIO__USE_AWS_ROLE=true` |
| Elasticsearch or OpenSearch | `ELASTICSEARCH__USERNAME`, `ELASTICSEARCH__PASSWORD`, or a user and password written in `ELASTICSEARCH__URL` | The password is `changeme` or `admin`. Not checked with `OPENSEARCH__REGION`, nor with `ELASTICSEARCH__API_KEY` unless `ELASTICSEARCH__ENGINE_CLIENT=opensearch` |
| Redis | `REDIS__USERNAME`, `REDIS__PASSWORD`, and in sentinel mode `REDIS__SENTINEL_USERNAME`, `REDIS__SENTINEL_PASSWORD` | The password is `changeme` |

Only a default password counts: the user name is reported with it when it is a default too, such as `guest`, `elastic` or `default`. A service without a password, as the bundled Redis and Elasticsearch, is not reported. When a [CyberArk provider](#read-the-service-credentials-of-opencti-from-cyberark) fills a password, OpenCTI does not check the configured one; it checks the S3 keys once the provider has returned them. The same check covers the admin password of OpenCTI and its health check key: see [Sign in the first time and keep an emergency account](/deploy/configure/first-access/).

`APP__CREDENTIALS__SECURE_BY_DEFAULT` decides what OpenCTI does with a default value:

- `false`, the default: OpenCTI starts, and logs the warning `Default credentials are configured, change them` with the names of the settings, never their values. A later version refuses to start with them. xtm-docker sets `APP__APP_LOGS__LOGS_LEVEL=error`, which hides this warning.
- `true`: OpenCTI logs `Default credentials check failed` and stops.

In xtm-docker, OpenCTI takes the RabbitMQ and S3 credentials from `RABBITMQ_DEFAULT_USER`, `RABBITMQ_DEFAULT_PASS`, `MINIO_ROOT_USER` and `MINIO_ROOT_PASSWORD` of `.env`. The `init` command of [Install with Docker Compose](/deploy/install/docker-compose/) draws random values for them; `.env.sample` holds `changeme` for both passwords and for the RabbitMQ user. Once `.env` holds your own values, turn the check on, so that a default value can no longer reach production:

```yaml
services:
  opencti:
    environment:
      - APP__CREDENTIALS__SECURE_BY_DEFAULT=true
```

:::verify
`docker compose up -d opencti`, then `docker compose ps` shows `opencti` healthy. When it restarts in a loop instead, `docker compose logs opencti` names the settings to change, after `Default credentials check failed`.
:::
::::

## Read the service credentials of OpenCTI from CyberArk

When your policy forbids passwords in `.env`, OpenCTI can fetch the credentials of Elasticsearch, Redis and S3 storage from the CyberArk Central Credential Provider when it connects to them. Each service has its prefix: `ELASTICSEARCH`, `REDIS`, or `MINIO` for S3 storage.

| Variable | Default | Set it to |
| --- | --- | --- |
| `<PREFIX>__CREDENTIALS_PROVIDER__SELECTOR` | | `cyberark`. Without it, OpenCTI ignores the other variables of the service |
| `<PREFIX>__CREDENTIALS_PROVIDER__CYBERARK__URI` | | The Accounts endpoint of the Central Credential Provider, such as `https://cyberark.example.com/AIMWebService/api/Accounts` |
| `<PREFIX>__CREDENTIALS_PROVIDER__CYBERARK__APP_ID`, `__SAFE`, `__OBJECT` | | The application ID, safe and object of the account, sent as `AppID`, `Safe` and `Object` |
| `<PREFIX>__CREDENTIALS_PROVIDER__CYBERARK__FIELD_TARGETS` | | A JSON list: the setting that receives each value of the secret, in order. Without it, OpenCTI reads `<PREFIX>__CREDENTIALS_PROVIDER__FIELD_TARGETS`. With neither, the secret fills nothing |
| `<PREFIX>__CREDENTIALS_PROVIDER__CYBERARK__DEFAULT_SPLITTER` | `:` | The separator between the values of the secret, as in `user:password`. OpenCTI cuts the secret at every separator, so a password that contains it is cut too: choose a separator your passwords never contain |
| `<PREFIX>__CREDENTIALS_PROVIDER__HTTPS_CERT__CRT`, `__KEY` | | When the provider asks for a client certificate: the certificate and its private key, each as its PEM content encoded in base64 on one line, not a path |
| `<PREFIX>__CREDENTIALS_PROVIDER__HTTPS_CERT__CA` | | Read only together with a client certificate. Leave it unset, and give the CA that signed the certificate of the provider to `NODE_EXTRA_CA_CERTS`, as described in [Trust your certificate authority and proxy](/deploy/configure/certificates/): a list set here replaces the CAs OpenCTI trusts, and does not load as a CA |
| `<PREFIX>__CREDENTIALS_PROVIDER__HTTPS_CERT__REJECT_UNAUTHORIZED` | off | `true` to check the certificate of the provider. Keep `true` in production: with `false`, anyone between OpenCTI and the provider can pose as it and read the secrets |

The settings a secret can fill depend on the service:

| Service | Prefix | Field targets |
| --- | --- | --- |
| Elasticsearch or OpenSearch | `ELASTICSEARCH` | `username` and `password`, or `apiKey` |
| Redis | `REDIS` | `username` and `password`; with Sentinel, also `sentinelUsername` and `sentinelPassword` |
| S3 storage | `MINIO` | `accessKeyId` and `secretAccessKey` |

For example, Example CERT keeps the Elasticsearch and Redis accounts of OpenCTI in the CyberArk safe `cti-platform`, each as its user name and password separated by `|`, because the passwords may contain a colon. Its Central Credential Provider accepts only clients with a certificate, signed like the provider's own certificate by the internal CA. The platform team stores the client certificate and its key, base64-encoded, in `.env` as `CYBERARK_CLIENT_CERT_B64` and `CYBERARK_CLIENT_KEY_B64` (for example with `base64 -w0 opencti-client.crt`), and adds to the `opencti` service in `docker-compose.override.yml`:

```yaml
services:
  opencti:
    volumes:
      - ./certs:/opt/opencti/certs:ro
    environment:
      - ELASTICSEARCH__CREDENTIALS_PROVIDER__SELECTOR=cyberark
      - ELASTICSEARCH__CREDENTIALS_PROVIDER__CYBERARK__URI=https://cyberark.example.com/AIMWebService/api/Accounts
      - ELASTICSEARCH__CREDENTIALS_PROVIDER__CYBERARK__APP_ID=opencti
      - ELASTICSEARCH__CREDENTIALS_PROVIDER__CYBERARK__SAFE=cti-platform
      - ELASTICSEARCH__CREDENTIALS_PROVIDER__CYBERARK__OBJECT=elasticsearch-opencti
      - 'ELASTICSEARCH__CREDENTIALS_PROVIDER__CYBERARK__FIELD_TARGETS=["username","password"]'
      - ELASTICSEARCH__CREDENTIALS_PROVIDER__CYBERARK__DEFAULT_SPLITTER=|
      - ELASTICSEARCH__CREDENTIALS_PROVIDER__HTTPS_CERT__CRT=${CYBERARK_CLIENT_CERT_B64}
      - ELASTICSEARCH__CREDENTIALS_PROVIDER__HTTPS_CERT__KEY=${CYBERARK_CLIENT_KEY_B64}
      - ELASTICSEARCH__CREDENTIALS_PROVIDER__HTTPS_CERT__REJECT_UNAUTHORIZED=true
      - REDIS__CREDENTIALS_PROVIDER__SELECTOR=cyberark
      - REDIS__CREDENTIALS_PROVIDER__CYBERARK__URI=https://cyberark.example.com/AIMWebService/api/Accounts
      - REDIS__CREDENTIALS_PROVIDER__CYBERARK__APP_ID=opencti
      - REDIS__CREDENTIALS_PROVIDER__CYBERARK__SAFE=cti-platform
      - REDIS__CREDENTIALS_PROVIDER__CYBERARK__OBJECT=redis-opencti
      - 'REDIS__CREDENTIALS_PROVIDER__FIELD_TARGETS=["username","password"]'
      - REDIS__CREDENTIALS_PROVIDER__CYBERARK__DEFAULT_SPLITTER=|
      - REDIS__CREDENTIALS_PROVIDER__HTTPS_CERT__CRT=${CYBERARK_CLIENT_CERT_B64}
      - REDIS__CREDENTIALS_PROVIDER__HTTPS_CERT__KEY=${CYBERARK_CLIENT_KEY_B64}
      - REDIS__CREDENTIALS_PROVIDER__HTTPS_CERT__REJECT_UNAUTHORIZED=true
      - NODE_EXTRA_CA_CERTS=/opt/opencti/certs/internal-ca.pem
```

Each service carries its own provider settings, client certificate included: Redis reads none of the Elasticsearch ones. `ELASTICSEARCH__CREDENTIALS_PROVIDER__HTTPS_CERT__CA` and its Redis counterpart stay unset: `NODE_EXTRA_CA_CERTS` makes OpenCTI trust the internal CA that signed the provider's certificate. Redis takes its field targets from the variable without `CYBERARK`, which serves any provider; both forms work.

OpenCTI logs `Remote credentials successfully fetched` at the `info` level, once for the source `elasticsearch`, and for the source `redis` at each Redis connection it opens. xtm-docker sets the `error` level: set `APP__APP_LOGS__LOGS_LEVEL=info` to see these lines. When the provider cannot be reached, the logs show `Remote credentials data fail to fetch, fallback`, at the `error` level. Then, as when the provider returns no secret, OpenCTI connects with the `ELASTICSEARCH__USERNAME` and `ELASTICSEARCH__PASSWORD` of its configuration, if any. The secrets of single sign-on providers use the same mechanism under another prefix: see [Authentication](/opencti/administer/authentications/).

## Check that it works

From the host, ask OpenCTI which versions of its services it reaches:

```bash
set -a; . ./.env; set +a
curl -s "http://localhost:$OPENCTI_PORT/graphql" \
  -H "Authorization: Bearer $OPENCTI_ADMIN_TOKEN" -H 'Content-Type: application/json' \
  -d '{"query":"{ about { dependencies { name version } } }"}'
```

:::verify
The answer lists `Search engine`, `RabbitMQ` and `Redis`, each with a version. `docker compose ps` shows `opencti`, `openaev` and `xtm-one` healthy.
:::

## What's next

- [Trust your certificate authority and proxy](/deploy/configure/certificates/), to trust the certificates of your services.
- [Connect the modules to each other](/deploy/configure/connect-modules/).
- [Back up and restore](/deploy/operate/backup/), for the data these services hold.
- [Scale: workers, platforms and clusters](/deploy/operate/scale/).
