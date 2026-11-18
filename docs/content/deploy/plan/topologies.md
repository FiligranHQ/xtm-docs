---
title: "Topologies: one host, a cluster, Kubernetes"
description: Where the modules and shared services of the XTM stack can run, from one host with Docker Compose to clusters and Kubernetes, and what each choice gives.
doc_type: explanation
products:
  - suite
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
sidebar_position: 5
---

The modules and shared services of the XTM stack can all run on one host, or be spread over several machines, clusters and managed services. This page explains the topologies the XTM documentation covers today, what each gives and where to go next. It does not replace [Sizing](/deploy/plan/sizing/), which gives the resources of each component.

::diagram{id=deploy-plan-topologies}

## The choices at a glance

| Topology | What runs where | Good for | Covered by |
| --- | --- | --- | --- |
| One host | Every module and shared service in Docker Compose, on one machine | Trying the platform, development, small deployments | [FiligranHQ/xtm-docker](https://github.com/FiligranHQ/xtm-docker) |
| Separate shared services | The modules in containers; the shared services in their own clusters or as managed cloud services | Production, when the data must outlive one machine | [Configure the shared services](/deploy/configure/shared-services/) |
| Cluster | Several platform instances and workers behind a load balancer, on clustered shared services | High availability and higher throughput | [Scale: workers, platforms and clusters](/deploy/operate/scale/) |
| Kubernetes | The containers of the stack on a Kubernetes cluster | Teams that run their services on Kubernetes | Not supported yet: [Install on Kubernetes](/deploy/install/kubernetes/) is coming soon |

## One host

The official deployment, xtm-docker, runs the whole stack on one machine with Docker Compose: XTM One, OpenCTI, OpenAEV, their default integrations, XTM Composer and one instance of each shared service. It is the fastest way to start, and the [quick start](/deploy/quick-start/) uses it.

Everything runs once. Elasticsearch runs as a single node, and OpenCTI creates its indices without replicas (`ELASTICSEARCH__NUMBER_OF_REPLICAS=0`). If the machine stops, the whole stack stops. The xtm-docker README asks for 16 GB of RAM at least and 32 GB for production.

## Separate shared services

For production, Filigran recommends running OpenCTI and all its dependencies in containers, on native cloud services or an orchestrator such as Kubernetes. The shared services are the part to move first: they hold all the data, while the platforms, workers and integrations keep none on disk.

Each shared service can be a cluster you run or a managed service:

| Shared service | OpenCTI also works with | OpenAEV also works with |
| --- | --- | --- |
| Elasticsearch | OpenSearch, and the AWS, GCP and Azure search services based on the Elasticsearch query language | Elasticsearch 9 and OpenSearch, detected from the URL (3.261029.0) |
| Redis | ElastiCache, Memorystore, and the AWS, GCP and Azure services based on the Redis protocol | Not used |
| RabbitMQ | Amazon MQ, CloudAMQP, and the AWS, GCP and Azure services based on AMQP | No alternative documented |
| S3 storage | Any service that follows the S3 standard | No alternative documented |
| PostgreSQL | Not used | No alternative documented |

XTM One needs PostgreSQL with the pgvector extension, Redis and S3 storage. No managed-service alternatives are documented for it yet.

## Cluster

OpenCTI scales horizontally: several platform instances connected to the same shared services, behind a load balancer, with more workers. Elasticsearch, Redis, RabbitMQ and S3 storage can each run in cluster mode. How to configure each of them, and how to split the platforms between users and ingestion, is in [Scale: workers, platforms and clusters](/deploy/operate/scale/).

Clustering of OpenAEV and XTM One is not documented yet.

## Kubernetes

Installing the platform on Kubernetes is not supported yet: xtm-docker targets Docker Compose only, and [Install on Kubernetes](/deploy/install/kubernetes/) is coming soon.

XTM Composer can start integrations on Kubernetes as well as on Docker: its daemon selector accepts `kubernetes`, `docker`, `podman`, `swarm` or `portainer`. xtm-docker sets `docker`. See [Configure XTM Composer](/deploy/integrations/configure-composer/).

For OpenCTI and OpenAEV, Helm charts are available as community contributions: [devops-ia/helm-opencti](https://github.com/devops-ia/helm-opencti) and [devops-ia/helm-openaev](https://github.com/devops-ia/helm-openaev). Filigran does not maintain them, and they are not part of xtm-docker.

## What's next

- [Sizing](/deploy/plan/sizing/): the resources of each component.
- [Install with Docker Compose](/deploy/install/docker-compose/).
- [Scale: workers, platforms and clusters](/deploy/operate/scale/).
