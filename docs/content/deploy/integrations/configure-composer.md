---
title: Configure XTM Composer
description: Set the identity, key, platforms and container engine of XTM Composer, with a YAML file or environment variables, and the options each engine applies to the integration containers.
doc_type: how-to
products:
  - composer
  - opencti
  - openaev
personas: [platform-engineer]
edition: ee
owner: docs-platform
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/deployment/integration-manager/configuration/
  - openaev:/deployment/ecosystem/integration-manager/configuration/
---

This page explains how XTM Composer reads its configuration and gives the settings you need to connect it to OpenCTI and OpenAEV and to your container engine. Every key, with its type and default, is in the XTM Composer sections of [OpenCTI integration settings](/reference/opencti/integration-settings/#xtm-composer-manager), generated from the code: they list the `opencti` keys, and the `openaev` keys are the same.

::diagram{id=deploy-integrations-configure-composer}

## Where settings come from

XTM Composer merges three sources, the last one winning:

1. `config/default.yaml`, shipped in the image at `/opt/xtm-composer/config/default.yaml`.
2. `config/<COMPOSER_ENV>.yaml`, if it exists. `COMPOSER_ENV` is `production` by default, so the file is `/opt/xtm-composer/config/production.yaml`.
3. Environment variables. A variable is the path of the key in capitals, with `__` between levels: `manager.logger.level` is `MANAGER__LOGGER__LEVEL`, `opencti.daemon.docker.network_mode` is `OPENCTI__DAEMON__DOCKER__NETWORK_MODE`.

To use a file in a container, mount that one file, not the folder: `default.yaml` must stay in place.

```bash
docker run -d --name xtm-composer \
  -v "$PWD/production.yaml:/opt/xtm-composer/config/production.yaml:ro" \
  -v "$PWD/keys:/keys:ro" \
  -v /var/run/docker.sock:/var/run/docker.sock \
  filigran/xtm-composer:latest
```

Lists, such as `extra_hosts` or `cap_add`, go in the YAML file. A YAML file does not expand variables: write the values, or give secrets as environment variables.

## XTM Composer itself

| Key | Environment variable | Default | Set it to |
| --- | --- | --- | --- |
| `manager.id` | `MANAGER__ID` | `default-manager-id` | A stable identifier, such as a UUID. The platforms know XTM Composer by it, and it labels every container XTM Composer manages |
| `manager.name` | `MANAGER__NAME` | `Filigran connector manager` | The name the platforms show |
| `manager.credentials_key_filepath` | `MANAGER__CREDENTIALS_KEY_FILEPATH` | | The path of the 4096-bit RSA private key, PKCS#8 PEM. Wins over `credentials_key` when both are set |
| `manager.credentials_key` | `MANAGER__CREDENTIALS_KEY` | | The key itself, when you cannot mount a file |
| `manager.execute_schedule` | `MANAGER__EXECUTE_SCHEDULE` | `10` | Seconds between two reconciliations |
| `manager.ping_alive_schedule` | `MANAGER__PING_ALIVE_SCHEDULE` | `60` | Seconds between two pings to each platform |
| `manager.image_tag_suffix` | `MANAGER__IMAGE_TAG_SUFFIX` | None, and `-fips` in the FIPS image of XTM Composer | `-fips` or `-ubi9`, to run the FIPS or Red Hat Universal Base Image (UBI) builds of the integrations. Another value stops XTM Composer at start |
| `manager.logger.level` | `MANAGER__LOGGER__LEVEL` | `info` | `trace`, `debug`, `info`, `warn` or `error` |
| `manager.logger.format` | `MANAGER__LOGGER__FORMAT` | `json` | `json` or `pretty` |

One of `credentials_key_filepath` and `credentials_key` is required: XTM Composer stops at start without a key. When it serves OpenAEV alone, it reads the key only at its first registration, and logs the error then.

XTM Composer adds `manager.image_tag_suffix` to the tag of every integration image it deploys, for both platforms and on every container engine:

- `opencti/connector-mitre:6.9.0` becomes `opencti/connector-mitre:6.9.0-fips`, and an image without a tag gets `latest-fips`.
- A tag that already ends with the suffix keeps it once, and an image pinned by digest (`@sha256:…`) stays as it is.
- With a private registry, the suffix goes after the tag: `registry.example.com/opencti/connector-mitre:6.9.0-fips`.

Every integration you deploy needs an image with that tag. OpenAEV publishes no FIPS images, as [Install offline or with FIPS 140-3](/deploy/install/air-gapped/) says. When the image cannot be pulled, XTM Composer deploys nothing and logs the reason on the integration in the platform: see [Troubleshoot XTM Composer](/deploy/integrations/troubleshoot/). An empty value turns the suffix off, in the FIPS image too: `MANAGER__IMAGE_TAG_SUFFIX=""`.

Keep `manager.logger.directory` and `manager.logger.console` at their default, `true`: XTM Composer then writes its logs to its standard output and to daily files in a `logs` folder next to its executable.

## The platforms

`opencti` and `openaev` take the same keys. Replace `OPENCTI__` with `OPENAEV__` for OpenAEV.

| Key | Environment variable | Default | Set it to |
| --- | --- | --- | --- |
| `opencti.enable` | `OPENCTI__ENABLE` | `true` (`false` for `openaev`) | `true` to serve this platform |
| `opencti.url` | `OPENCTI__URL` | `http://host.docker.internal:4000` | The URL of the platform, as XTM Composer and its integrations reach it. XTM Composer also gives it to every integration as `OPENCTI_URL` or `OPENAEV_URL` |
| `opencti.token` | `OPENCTI__TOKEN` | `ChangeMe` | An API token of the platform |
| `opencti.unsecured_certificate` | `OPENCTI__UNSECURED_CERTIFICATE` | `false` | `true` to accept an invalid or self-signed certificate on the platform. Not for production |
| `opencti.request_timeout` | `OPENCTI__REQUEST_TIMEOUT` | `30` | Seconds before a request to the platform times out |
| `opencti.connect_timeout` | `OPENCTI__CONNECT_TIMEOUT` | `10` | Seconds before a connection to the platform times out |
| `opencti.logs_schedule` | `OPENCTI__LOGS_SCHEDULE` | `10` | Seconds between two sends of the logs of an integration |

The proxy keys of each platform are in [Run integrations behind a proxy](/deploy/integrations/proxy/), the registry keys in [Pull integrations from a private registry](/deploy/integrations/private-registry/).

## The container engine

`opencti.daemon.selector`, or `OPENCTI__DAEMON__SELECTOR`, says where XTM Composer runs the integrations of the platform: `docker`, `podman`, `kubernetes` (the default), `swarm` or `portainer`. Each platform has its own `daemon` block, so OpenCTI and OpenAEV can use different engines.

The block of the selected engine shapes what XTM Composer creates for each integration: a container, a Swarm service or a Kubernetes Deployment. It sets the network the integration joins, how it resolves names, its Linux privileges and the resources it may use.

- The options apply to every integration of the platform alike. There is no value per integration.
- XTM Composer reads them when it creates a container. An integration already deployed gets a change only when its container is created again: change its configuration with **Update**, or remove its container.
- The keys are under `opencti.daemon.<engine>`, and under `openaev.daemon.<engine>` for OpenAEV. A key with one value also takes an environment variable, such as `OPENCTI__DAEMON__DOCKER__PRIVILEGED=false`. A list or a map, such as `cap_drop` or `sysctls`, goes in the YAML file.
- A key you leave out keeps the default of the engine.

[OpenCTI integration settings](/reference/opencti/integration-settings/#xtm-composer-orchestrator) gives the type of every key, engine by engine.

::::tabs
:::tab[Docker]
XTM Composer talks to the Docker daemon through its socket, mounted in the XTM Composer container, and starts each integration as a container next to it. The `opencti.daemon.docker` block is optional, and so is each of its keys: without them, the integrations get the defaults of Docker. The types are in [XTM Composer: Docker](/reference/opencti/integration-settings/#xtm-composer-docker).

| Key under `opencti.daemon.docker` | Effect on the integration containers | Default |
| --- | --- | --- |
| `socket_path` | The Docker API socket XTM Composer uses, as a path or a `unix://` URL | `/var/run/docker.sock` |
| `network_mode` | The network they join. Name the network of the platform and RabbitMQ, `xtm_default` in xtm-docker: on another network, an integration cannot reach the platform and fails at start. `host` and `none` also work | The `bridge` network of Docker |
| `extra_hosts` | Lines added to their `/etc/hosts`, as `host:ip`. `host-gateway` as the IP stands for the host | None |
| `dns` | The DNS servers they use | Those of the host |
| `dns_search` | The domains added to a short host name | Those of the host |
| `privileged` | `true` gives them every privilege on the host. Leave it off unless an integration documents the need | `false` |
| `cap_add` | Linux capabilities added, such as `NET_ADMIN` | The default set of Docker |
| `cap_drop` | Linux capabilities removed. `ALL` removes every one: test your integrations after the change | The default set of Docker |
| `security_opt` | Security options: `no-new-privileges:true`, a `seccomp=` or `apparmor=` profile, SELinux `label=` options | The profiles of Docker |
| `userns_mode` | User namespace. `host` turns off the remapping of a daemon set with `userns-remap` | The setting of the daemon |
| `pid_mode` | PID namespace. `host` lets them see the processes of the host | Their own namespace |
| `ipc_mode` | IPC namespace: `private`, `shareable` or `host` | The default of the daemon |
| `uts_mode` | UTS namespace. `host` gives them the host name of the host | Their own namespace |
| `runtime` | The OCI runtime declared in the daemon, such as `runsc` for gVisor | The default runtime, usually `runc` |
| `shm_size` | The size of `/dev/shm`, in bytes | 64 MB |
| `sysctls` | Kernel parameters of their namespaces, as a map: `net.ipv4.tcp_keepalive_time: "600"` | Those of the host |
| `ulimits` | Resource limits, each with a `name`, a `soft` and a `hard` value. An entry without all three is ignored | The limits of the daemon |

Example CERT runs OpenCTI with xtm-docker on one host. It hardens the integrations: no Linux capability, no privilege escalation, its internal DNS, and more open files for the large feeds it imports.

```yaml
opencti:
  daemon:
    selector: docker
    docker:
      network_mode: xtm_default
      extra_hosts:
        - "proxy.example.com:10.0.20.5"
      dns:
        - 10.0.0.53
      dns_search:
        - example.com
      privileged: false
      cap_drop:
        - ALL
      security_opt:
        - "no-new-privileges:true"
      shm_size: 134217728
      ulimits:
        - name: nofile
          soft: 65536
          hard: 65536
```
:::
:::tab[Podman]
XTM Composer drives Podman through its Docker-compatible API socket, rootful or rootless, on a Linux host only. The `opencti.daemon.podman` block takes the keys of the Docker tab, with the same effect. The block is optional, and so is each of its keys. The types are in [XTM Composer: Podman](/reference/opencti/integration-settings/#xtm-composer-podman).

| Key under `opencti.daemon.podman` | Effect on the integration containers | Default |
| --- | --- | --- |
| `socket_path` | The Podman API socket, as a path or a `unix://` URL. Another scheme stops XTM Composer at start | The first socket that exists: `$XDG_RUNTIME_DIR/podman/podman.sock`, then `/run/user/<uid>/podman/podman.sock` (not for root), then `/run/podman/podman.sock` |
| `network_mode` | The network they join, such as the network of the platform. Check it with `podman network inspect` | The default network of Podman |
| `extra_hosts`, `dns`, `dns_search`, `privileged`, `cap_add`, `cap_drop`, `security_opt`, `userns_mode`, `pid_mode`, `ipc_mode`, `uts_mode`, `runtime`, `shm_size`, `sysctls`, `ulimits` | As in the Docker tab | The defaults of Podman |

`XDG_RUNTIME_DIR` is read from the environment of XTM Composer, at each attempt to find the socket.

Example CERT runs a rootful Podman on a Red Hat Enterprise Linux host, with the socket mounted in the XTM Composer container at the default path:

```yaml
opencti:
  daemon:
    selector: podman
    podman:
      socket_path: /run/podman/podman.sock
      network_mode: xtm_default
      cap_drop:
        - ALL
      security_opt:
        - "no-new-privileges:true"
      ulimits:
        - name: nofile
          soft: 65536
          hard: 65536
```
:::
:::tab[Kubernetes]
XTM Composer uses the service account of its pod, or the current kubeconfig context. It creates one Deployment per integration in its own namespace, with one replica when the integration is started and none when it is stopped. The default configuration holds the block, so the `kubernetes` selector works with no key. The types are in [XTM Composer: Kubernetes](/reference/opencti/integration-settings/#xtm-composer-kubernetes).

| Key under `opencti.daemon.kubernetes` | Effect on the integration Deployments | Default |
| --- | --- | --- |
| `image_pull_policy` | When Kubernetes pulls the image: `Always`, `IfNotPresent` or `Never`. Another value logs a warning and falls back to `IfNotPresent` | `IfNotPresent` |
| `image_resources` | The requests and limits of every integration container, in the Kubernetes `resources` format. YAML only | None: the LimitRange of the namespace applies, if any |
| `base_deployment` | A Deployment manifest XTM Composer starts every integration Deployment from. What XTM Composer sets wins: name, labels, replicas, the integration container. The rest of the base is kept, such as pod annotations, `nodeSelector`, `tolerations`, `affinity` or a pod `securityContext`. YAML only | None |
| `base_deployment_json` | The same manifest as a JSON string, for an environment variable. Read only when `base_deployment` is not set. A string that is not valid JSON makes XTM Composer fail when it deploys an integration | None |

Example CERT keeps its integrations on the nodes it dedicates to them, with bounded resources:

```yaml
opencti:
  daemon:
    selector: kubernetes
    kubernetes:
      image_pull_policy: IfNotPresent
      image_resources:
        requests:
          cpu: 250m
          memory: 256Mi
        limits:
          cpu: "1"
          memory: 1Gi
      base_deployment:
        metadata:
          labels:
            team: example-cert
        spec:
          template:
            spec:
              nodeSelector:
                workload: integrations
              tolerations:
                - key: dedicated
                  operator: Equal
                  value: integrations
                  effect: NoSchedule
              securityContext:
                seccompProfile:
                  type: RuntimeDefault
```

Without a file, give the base as `OPENCTI__DAEMON__KUBERNETES__BASE_DEPLOYMENT_JSON='{"spec":{"template":{"spec":{"nodeSelector":{"workload":"integrations"}}}}}'`.
:::
:::tab[Docker Swarm]
XTM Composer talks to `/var/run/docker.sock` on a Swarm manager node: there is no socket key. It creates one replicated service per integration, with one replica when the integration is started and none when it is stopped. The XTM stack runs on Docker Compose: use this engine when your platform runs on a Swarm cluster of your own.

The `opencti.daemon.swarm` block is required with the `swarm` selector, even with a single key: without it, XTM Composer stops at start with `Missing swarm configuration`. Every key in it is optional. The types are in [XTM Composer: Docker Swarm](/reference/opencti/integration-settings/#xtm-composer-docker-swarm).

| Key under `opencti.daemon.swarm` | Effect on the integration services | Default |
| --- | --- | --- |
| `network` | The overlay network the services attach to. Name the network of the platform and RabbitMQ, or the integrations cannot reach them | None |
| `extra_hosts` | Lines of their `/etc/hosts`, written as in that file: `10.0.20.5 proxy.example.com`. XTM Composer passes them to Swarm as they are, so the `host:ip` form of Docker does not work here | None |
| `dns`, `dns_search` | The DNS servers and the search domains of the containers | Those of the node |
| `cap_add`, `cap_drop` | Linux capabilities added or removed | The default set of Docker |
| `sysctls` | Kernel parameters of their namespaces, as a map | Those of the node |
| `ulimits` | Resource limits, each with a `name`, a `soft` and a `hard` value. An entry without all three is ignored | The limits of the daemon |
| `hostname` | The host name of the containers. Every integration of the platform gets the same one | The container ID |
| `user` | The user the containers run as, as `uid:gid` or a name | The user of the image |
| `read_only` | `true` mounts the root filesystem read-only. An integration that writes files there fails | `false` |
| `init` | `true` runs an init process that forwards signals and reaps child processes | The default of the daemon |
| `stop_grace_period` | The time a container gets to stop before it is killed, in nanoseconds: `10000000000` is 10 seconds | 10 seconds |
| `resources.cpu_limit` | The CPU a service may use, in nano CPUs: `1000000000` is one CPU | No limit |
| `resources.memory_limit` | The memory a service may use, in bytes. Above it, the container is killed | No limit |
| `resources.cpu_reservation` | The CPU a node must have free to run the service, in nano CPUs | None |
| `resources.memory_reservation` | The memory a node must have free to run the service, in bytes | None |
| `placement_constraints` | The nodes the services may run on, such as `node.role==worker` or `node.labels.integrations==true` | Any node |
| `placement_preferences` | Labels to spread the services across, such as `node.labels.zone` | None |
| `restart_condition` | When Swarm restarts a container: `none`, `on-failure` or `any`. Another value counts as `any` | `any` |
| `restart_delay` | The wait between two restarts, in nanoseconds | 5 seconds |
| `restart_max_attempts` | The maximum number of restarts. `0` sets no limit | No limit |

Example CERT runs OpenCTI on a Swarm cluster of three zones, with the overlay network `xtm_backend`. It runs the integrations on labelled worker nodes, spread across zones, within one CPU and 1 GB each:

```yaml
opencti:
  daemon:
    selector: swarm
    swarm:
      network: xtm_backend
      extra_hosts:
        - "10.0.20.5 proxy.example.com"
      dns_search:
        - example.com
      cap_drop:
        - ALL
      init: true
      stop_grace_period: 30000000000
      resources:
        cpu_limit: 1000000000
        memory_limit: 1073741824
        cpu_reservation: 250000000
        memory_reservation: 268435456
      placement_constraints:
        - node.role==worker
        - node.labels.integrations==true
      placement_preferences:
        - node.labels.zone
      restart_condition: any
      restart_delay: 10000000000
      restart_max_attempts: 0
```
:::
:::tab[Portainer]
XTM Composer creates the integration containers through the Portainer API, in one Docker environment of Portainer. It calls the Docker API that Portainer exposes for that environment, with the API key in the `X-API-KEY` header, and does not check the certificate of Portainer. Only `network_mode` and `stack` shape the containers: the container options of the Docker tab do not apply. The types are in [XTM Composer: Portainer](/reference/opencti/integration-settings/#xtm-composer-portainer).

| Key under `opencti.daemon.portainer` | Effect | Default |
| --- | --- | --- |
| `api` | The Portainer URL, with its port | `https://host.docker.internal:9443`, and `https://localhost:9443` for OpenAEV |
| `api_key` | A Portainer access token of a user who manages containers in the environment. Give it as an environment variable | `ChangeMe` |
| `env_id` | The ID of the environment: the number in its address in Portainer | `3` |
| `env_type` | `docker`, the only type supported. Another value stops XTM Composer at start | `docker` |
| `api_version` | The Docker API version of the calls. Lower it for an older Docker Engine | `v1.44` |
| `stack` | A name set as the `com.docker.compose.project` label of each container, so that Portainer shows them in that stack | None, and `opencti-dev` for OpenAEV |
| `network_mode` | The network the containers join, such as the network of the platform | None, and `opencti-dev_default` for OpenAEV |

The default configuration gives the `openaev` block `stack: opencti-dev` and `network_mode: opencti-dev_default`. When OpenAEV uses Portainer, set both: otherwise its integrations try to join a network that does not exist.

Example CERT manages its Docker host in Portainer, as environment 2, with the token given as `OPENCTI__DAEMON__PORTAINER__API_KEY`:

```yaml
opencti:
  daemon:
    selector: portainer
    portainer:
      api: https://portainer.example.com:9443
      env_id: "2"
      env_type: docker
      api_version: v1.44
      stack: xtm
      network_mode: xtm_default
```
:::
::::

## Show what an integration receives

To see the environment XTM Composer gives each container it starts, set `MANAGER__DEBUG__SHOW_ENV_VARS=true`: it logs a `Starting connector` line with the variables, secrets replaced by `***REDACTED***`. `MANAGER__DEBUG__SHOW_SENSITIVE_ENV_VARS=true` shows the secrets too: use it only while you debug, then turn it off.

## A complete example

:::example[Serve both platforms of Example CERT from one Docker host]
Example CERT runs OpenCTI and OpenAEV with xtm-docker. `/opt/xtm-composer/config/production.yaml` for an XTM Composer that serves both platforms on Docker, with the key mounted at `/keys/private_key.pem` and the tokens given as `OPENCTI__TOKEN` and `OPENAEV__TOKEN`:

```yaml
manager:
  id: 8215614c-7139-422e-b825-b20fd2a13a23
  name: XTM Composer
  credentials_key_filepath: /keys/private_key.pem
opencti:
  enable: true
  url: http://opencti:8080
  daemon:
    selector: docker
    docker:
      network_mode: xtm_default
openaev:
  enable: true
  url: http://openaev:8080
  daemon:
    selector: docker
    docker:
      network_mode: xtm_default
```

Replace the `id` with a UUID of your own, generated once with `python3 -c 'import uuid; print(uuid.uuid4())'`. XTM Composer then registers with both platforms and starts their integrations on the `xtm_default` network.
:::

## Check that it works

Restart XTM Composer and read its first lines: it logs `Starting XTM composer` with its version and its `image_tag_suffix`, then `Successfully loaded RSA private key (PKCS#8 format)`, then `Manager registered` for each platform it serves. A setting it cannot read stops it at start, with an error that names the setting.

## What's next

- [OpenCTI integration settings](/reference/opencti/integration-settings/#xtm-composer-manager), every key of XTM Composer with its type and default.
- [Run integrations behind a proxy](/deploy/integrations/proxy/) and [Pull integrations from a private registry](/deploy/integrations/private-registry/).
- [Troubleshoot XTM Composer](/deploy/integrations/troubleshoot/).
