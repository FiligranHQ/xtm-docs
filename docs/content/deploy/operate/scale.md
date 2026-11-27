---
title: "Scale: workers, platforms and clusters"
description: Add OpenCTI workers in the XTM stack, run several OpenCTI nodes as a cluster, and put Elasticsearch, Redis, RabbitMQ and S3 storage in cluster mode.
doc_type: how-to
products:
  - opencti
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/advanced/clustering/
---

OpenCTI scales horizontally: you add workers to write data faster, OpenCTI nodes to serve more users and more ingestion, and you put its shared services in cluster mode for capacity and high availability. This page goes from the first step, more workers in xtm-docker, to a full cluster.

## Before you start

- A running OpenCTI: see [Install with Docker Compose](/deploy/install/docker-compose/).
- To size what you add, see [Sizing](/deploy/plan/sizing/).

## Add workers

Workers take the bundles that connectors put in RabbitMQ and write them through the OpenCTI API. More workers write more bundles in parallel. xtm-docker starts three, as the `worker` service with `replicas: 3`.

:::steps
1. Set the number of workers in `docker-compose.override.yml`, as [Change the stack without editing docker-compose.yml](/deploy/install/docker-compose/#change-the-stack-without-editing-docker-compose-yml) describes. For four workers:

   ```yaml
   services:
     worker:
       deploy:
         replicas: 4
   ```

2. Apply it.

   ```bash
   docker compose up -d worker
   ```

   :::verify
   ```bash
   docker compose ps worker --format '{{.Name}}'
   ```
   It lists four containers.
   :::
:::

To try a number without changing a file, run `docker compose up -d --scale worker=4`. The next `docker compose up -d` without `--scale` goes back to the number in the files.

Keep three or four workers for each OpenCTI node. Past that, workers wait for the platform and can fail with lock timeouts: add an OpenCTI node instead.

### Size what each worker does at once

A worker reads every connector queue at once, but writes one bundle at a time from each queue, in a pool of threads. The pools cap how many bundles a worker writes in parallel. Set these variables on the `worker` service; without containers, they can also go in the `config.yml` next to `worker.py`, under `opencti` or `worker` as their key says.

| Variable | `config.yml` key | Default | Effect |
| --- | --- | --- | --- |
| `OPENCTI_EXECUTION_POOL_SIZE` | `opencti.execution_pool_size` | `2` | Threads that write the bundles of the connectors |
| `OPENCTI_REALTIME_EXECUTION_POOL_SIZE` | `opencti.realtime_execution_pool_size` | `3` | Threads kept for the work users wait for: the validation of drafts, the form intakes and, with `PLAYBOOK_MANAGER__REALTIME_PRIORITY=true` on the platform, the playbooks. When one pool is full and the other has a free thread, the worker borrows it, so a quiet worker writes up to five bundles at once by default |
| `WORKER_LISTEN_POOL_SIZE` | `worker.listen_pool_size` | `5` | Threads that call the connectors that receive their work over HTTP rather than from RabbitMQ: see [Run an integration by hand](/deploy/integrations/manual/) |

Each thread holds a call to the platform, so a node counts threads, not workers: two workers with four threads load it as much as four workers with two. Raise the pools rather than the number of workers when the host of the workers has CPU left and the platform does not: the three or four workers per node of [Add workers](#add-workers) count with the default pools. The observability metrics of the workers show the threads: `opencti_running_ingestion_units` counts the threads the two pools have started, which stay once started, and `opencti_max_ingestion_units` the size of the connector pool only, so a busy worker shows more running units than its maximum. See [Observability metrics](/deploy/operate/metrics/).

### Shape the calls of the workers to the platform

When the workers reach OpenCTI from another network, through a reverse proxy or an API gateway, these settings shape their calls to the API:

| Variable | Default | Effect |
| --- | --- | --- |
| `OPENCTI_SSL_VERIFY` | `false` | Whether the worker checks the certificate of the platform. Set `true` as soon as `OPENCTI_URL` is an `https://` address: see [Trust your certificate authority and proxy](/deploy/configure/certificates/#trust-your-internal-certificate-authority) to trust your CA |
| `OPENCTI_CUSTOM_HEADERS` | | Headers added to the calls of the worker, as `name:value` pairs separated by `;`, such as `X-Request-Source:opencti-worker;X-Region:eu-west-3`. A pair without `:` is ignored, with `Ignored invalid header pair` in the logs |
| `OPENCTI_REQUESTS_TIMEOUT` | `300` | Seconds the worker waits for an answer of the API before it gives up on a call |
| `WORKER_PROMETHEUS_TELEMETRY_HOST` | `0.0.0.0` | The address the metrics endpoint of the worker listens on, with `WORKER_TELEMETRY_ENABLED=true`. `0.0.0.0` listens on every interface, as a container needs. Without containers, `127.0.0.1` keeps the metrics on the host |

:::warning
In this version, the headers and the timeout apply only to the calls the worker makes for itself, such as the list of connectors it reads every minute. The bundles it writes go without the headers and wait 300 seconds: a gateway that requires a header refuses them, so let the workers reach the platform without it, for example on an ingestion node of [Run OpenCTI as a cluster](#run-opencti-as-a-cluster). Set the timeout in `config.yml`, as `opencti.requests_timeout`, rather than as `OPENCTI_REQUESTS_TIMEOUT`: the worker passes the variable on as text, which its HTTP client refuses, and the worker then never reads the list of connectors.
:::

:::example[Workers in another data center over HTTPS]
Example CERT runs its workers in a second data center. They reach the ingestion node at `https://cti-ingest.example.com`, whose certificate the internal CA signed, and Prometheus scrapes each worker on the worker host:

```yaml
services:
  worker:
    environment:
      - OPENCTI_URL=https://cti-ingest.example.com
      - OPENCTI_TOKEN=${OPENCTI_WORKER_TOKEN}
      - OPENCTI_SSL_VERIFY=true
      - HTTPS_CA_CERTIFICATES=/opt/certs/internal-ca.pem
      - OPENCTI_EXECUTION_POOL_SIZE=4
      - OPENCTI_REALTIME_EXECUTION_POOL_SIZE=2
      - WORKER_TELEMETRY_ENABLED=true
      - WORKER_PROMETHEUS_TELEMETRY_HOST=0.0.0.0
    volumes:
      - ./certs:/opt/certs:ro
    deploy:
      replicas: 2
```

Two workers with four threads for the connectors write as many bundles at once as four workers with the default two, with half the containers to watch. They refuse any server at `cti-ingest.example.com` whose certificate the internal CA did not sign.
:::

### Tune what workers write

Two behaviors of the workers change what reaches the knowledge base. Set their variables on the `worker` service.

`WORKER_OBJECTS_MAX_REFS` (default `0`): when above 0, a worker does not write an object that has this many references or more, such as a report that contains 50,000 objects. It sets the object aside in the RabbitMQ queue `listen_too-large-bundle`, with the connector it came from. `0` writes every object. Without containers, it can also go in the `config.yml` next to `worker.py`, as `objects_max_refs` under `worker`.

When OpenCTI refuses an object because an object it refers to is not written yet, a worker tries again up to 4 times, waiting 1 to 3 seconds before each try.

:::only{opencti=">=7.261030"}
`OPENCTI_MISSING_REF_RETRY_EXPONENTIAL=true` changes those waits to about 0.5, 1, 2, then 4 seconds, which suits a reference that arrives a moment later. `OPENCTI_MISSING_REF_RETRY_INITIAL_DELAY` (`0.5` seconds) sets the first wait and `OPENCTI_MISSING_REF_RETRY_FACTOR` (`2`) multiplies it at each try; each wait varies by up to half its value. These three are read from the environment only.
:::

The log level and the metrics of workers are in [Troubleshoot](/deploy/operate/troubleshoot/) and [Observability metrics](/deploy/operate/metrics/), and their TLS settings for RabbitMQ in [Configure the shared services](/deploy/configure/shared-services/#rabbitmq).

## Run OpenCTI as a cluster

Several OpenCTI nodes form a cluster when they use the same Elasticsearch, Redis, RabbitMQ and S3 storage, with the same configuration. They share the work through Redis: you do not declare the nodes to each other. Once they run, :menu[Settings > Parameters] shows the **Number of node(s)**.

To avoid congestion, give the nodes two roles:

::diagram{id=deploy-operate-scale}

- **User nodes**, behind a load balancer, serve the interface, the API and the registration of connectors.
- **Ingestion nodes** receive the bundles of the workers. Point each worker at one of them with `OPENCTI_URL`, three or four workers per node. Users never reach these nodes.

The rule engine, the task manager and the notification manager use resources in the process of a node. Turn them off on the user nodes; OpenCTI runs each manager on the nodes where it is still on.

```yaml
- RULE_ENGINE__ENABLED=false
- TASK_SCHEDULER__ENABLED=false
- NOTIFICATION_MANAGER__ENABLED=false
```

What each manager does is in [How OpenCTI works](/opencti/concepts/architecture/#managers). Each node needs its health probes behind the load balancer: see [Check that the platforms are healthy](/deploy/operate/health-probes/).

These settings shape each node:

| Variable | Default | Set it to |
| --- | --- | --- |
| `APP__PORT` | `4000` | The port the node listens on. xtm-docker sets `8080`: the workers and the load balancer reach the nodes on it |
| `APP__ENABLED_UI` | `true` | `false` on the ingestion nodes: they serve the API only, without the web interface and the GraphQL playground |
| `APP__ENABLED` | `true` | `false` on a node that only runs managers. It starts no HTTP server, so it serves neither API nor health probes: watch its process or its logs instead |
| `APP__NODE_IDENTIFIER` | A random UUID at each start | A name of its own for each node, such as `opencti-ingest-1`, so that it keeps the same identity in the cluster across restarts. Give each node a different one |

Every node of the cluster runs the same platform: they share its identifier, kept in the platform settings. `PLATFORM_ID` forces that identifier, a UUID, when OpenCTI creates the platform or, on an existing one, at the next start. Use it to rebuild a platform with the identifier it had, such as for an Enterprise Edition license issued for that platform; leave it unset otherwise.

### Locks between the nodes

Before it writes an element, a node takes a lock on it in Redis, so that two nodes or two workers never write the same element at once. A node holds a lock for `APP__CONCURRENCY__MAX_TTL` and extends it while it works; a lock its node stops extending, after a crash, frees itself after that time. When the element is already locked, the node tries again `APP__CONCURRENCY__RETRY_COUNT` times, every `APP__CONCURRENCY__RETRY_DELAY` give or take a random `APP__CONCURRENCY__RETRY_JITTER`, then fails with a lock error.

| Variable | Default | Effect |
| --- | --- | --- |
| `APP__CONCURRENCY__RETRY_COUNT`, `APP__CONCURRENCY__RETRY_DELAY`, `APP__CONCURRENCY__RETRY_JITTER` | `100`, `250` ms, `100` ms | How long a node waits for a locked element: about 25 seconds by default |
| `APP__CONCURRENCY__MAX_TTL` | `60000` ms | The lifetime of a lock, and of each extension |
| `APP__CONCURRENCY__EXTENSION_THRESHOLD` | `5000` ms | The node extends a lock when it is twice this time away from expiring. Raise it if locks expire while a busy node works |
| `APP__CHILD_LOCKING_PROCESS__ENABLED` | `true` | Takes and extends the locks in a separate Node.js process, so that a busy node does not miss an extension. Keep `true` |
| `APP__CHILD_LOCKING_PROCESS__MAX_MEMORY` | `256` MB | The heap of that process |

Lock errors in the logs of the workers mean that they wait too long for the same elements: keep three or four workers per node, as [Add workers](#add-workers) says, and add an ingestion node rather than workers.

:::only{opencti=">=7.261030"}
Two [observability metrics](/deploy/operate/metrics/) show the contention before the errors: `opencti_lock_contentions` counts the locks that needed more than one attempt, and `opencti_lock_acquire_wait` measures the wait, in milliseconds.
:::

All settings: [Technical customization](/reference/opencti/configuration/#technical-customization).

## Put the shared services in cluster mode

### Elasticsearch

Declare every node of the cluster, as a JSON array:

```yaml
- 'ELASTICSEARCH__URL=["https://user:pass@node1:9200","https://user:pass@node2:9200"]'
```

OpenCTI also works with OpenSearch and with the managed search services of AWS, Google Cloud and Azure that are based on Elasticsearch or OpenSearch.

### Redis

Turn on cluster mode and list the nodes:

```yaml
- REDIS__MODE=cluster
- 'REDIS__HOSTNAMES=["node1:6379","node2:6379"]'
```

:::only{opencti=">=7.261030"}
With TLS (`REDIS__USE_SSL=true`), OpenCTI checks the certificate of every node against `REDIS__HOSTNAME` by default. Set `REDIS__TLS_SERVERNAME` when all the node certificates share one name, such as a cluster host name or a wildcard. Set `REDIS__TLS_CLUSTER_NODE_MODE=true` to check each node's certificate against that node's own name.
:::

OpenCTI also works with ElastiCache, Memorystore and the other managed services that speak the Redis protocol.

### RabbitMQ

OpenCTI connects to one RabbitMQ host. For a RabbitMQ cluster, put a TCP load balancer in front of its nodes and point OpenCTI at it:

```yaml
- RABBITMQ__HOSTNAME=load-balancer-rabbitmq
```

OpenCTI also works with Amazon MQ, CloudAMQP and the other managed services that speak AMQP.

### S3 storage

OpenCTI works with any storage that follows the S3 standard. [RustFS](https://github.com/rustfs/rustfs), the S3 server of xtm-docker, can itself run as a cluster of several nodes.

## What's next

- [Sizing](/deploy/plan/sizing/), to choose the number of nodes and workers.
- [Manage Elasticsearch indices](/deploy/operate/rollover/), to keep the Elasticsearch shards at a reasonable size.
- [Observability metrics](/deploy/operate/metrics/), to watch the load of each node.
