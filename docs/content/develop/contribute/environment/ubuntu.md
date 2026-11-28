---
title: Prepare an Ubuntu development environment
sidebar_label: Environment on Ubuntu
description: Install on Ubuntu the container engine, Node.js, Yarn, Python and Java versions that the OpenCTI and OpenAEV source code requires.
doc_type: how-to
products:
  - opencti
  - openaev
personas: [developer]
owner: docs-platform
last_reviewed: 2026-11-01
sidebar_position: 1
no_screenshot: "This page installs command-line tools on a workstation: there is no module screen to show."
migrated_from:
  - opencti:/development/environment_ubuntu/
  - openaev:/development/environment-ubuntu/
---

This page installs on Ubuntu the tools you need to run OpenCTI or OpenAEV from source: a container engine for their services, and the exact Node.js, Yarn, Python and Java versions their code requires. It is written for Ubuntu 24.04 LTS. When the tools are in place, continue with [Build and run from source](/develop/contribute/build-from-source/).

::diagram{id=develop-contribute-build-from-source}

## Before you start

- Ubuntu 24.04 LTS, with `sudo` rights. Ubuntu 24.04 ships Python 3.12, the version the OpenCTI images and its continuous integration (CI) use.
- At least 8 GB of memory for OpenCTI: its development Elasticsearch alone starts with a 2 GB Java heap (`ES_JAVA_OPTS=-Xms2G -Xmx2G` in `opencti-platform/opencti-dev/docker-compose.yml`).
- The versions below come from the code of OpenCTI 7.261030 and OpenAEV 3.261029. Contributions go to the default branches, `master` for OpenCTI and `main` for OpenAEV. The LTS branches of OpenCTI use older tools.

| Tool | OpenCTI | OpenAEV | Where the code sets it |
| --- | --- | --- | --- |
| Node.js | 22.18.0 or later; the images and CI run Node.js 22 | 24.11.0 or later | `engines` in `package.json` |
| Yarn | 4.18.1, through Corepack | 4.18.1, through Corepack | `packageManager` in `package.json` |
| Python | 3.12 | Not needed | Images `node:22-alpine3.23` and `python:3.12-alpine` |
| Java | Not needed | 21 | `java.version` in `pom.xml` |
| Maven | Not needed | 3.9 or later | OpenAEV development guide |
| Container engine | Docker or Podman, with Compose | Docker or Podman, with Compose | `opencti-dev` and `openaev-dev` |

## Install the shared tools

:::::steps
1. Install Git, the C and C++ toolchain, and Python with its `venv` and header packages:

   ```bash
   sudo apt-get update
   sudo apt-get install -y git curl build-essential python3 python3-venv python3-dev
   ```

   OpenCTI compiles two native Node.js modules at install time, `re2` and `node-calls-python`: they need `build-essential` and `python3-dev`. Ubuntu 24.04 refuses `pip install` outside a virtual environment, so you need `python3-venv`.

2. Install a container engine with Compose. Pick one.

   ::::tabs
   :::tab[Docker]
   Install Docker Engine and its Compose plugin from the Ubuntu archive, then let your user run Docker without `sudo`:

   ```bash
   sudo apt-get install -y docker.io docker-compose-v2
   sudo usermod -aG docker "$USER"
   ```

   Log out and back in for the group to apply. Docker's own repository, described in [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/), gives more recent versions.
   :::
   :::tab[Podman]
   Install Podman and a Compose provider:

   ```bash
   sudo apt-get install -y podman podman-compose
   ```

   The development stacks then start with `podman compose up -d`.
   :::
   ::::

3. Raise the memory map limit that Elasticsearch needs, now and at every boot:

   ```bash
   echo 'vm.max_map_count=262144' | sudo tee /etc/sysctl.d/99-elasticsearch.conf
   sudo sysctl --system
   ```

   Without it, the Elasticsearch container of the development stack can fail to start or stop under load.

4. Install nvm, the Node.js version manager, with the install script of [its README](https://github.com/nvm-sh/nvm/blob/master/README.md#installing-and-updating), then open a new terminal.
:::::

## Install the tools for OpenCTI

:::steps
1. Install Node.js 22 and make it the default:

   ```bash
   nvm install 22
   nvm alias default 22
   ```

2. Remove Yarn 1 if a global one is installed, then install Corepack. Corepack fetches the Yarn version that `package.json` pins when you run `yarn` in the repository:

   ```bash
   npm uninstall -g yarn
   npm install -g corepack
   ```

   nvm keeps global packages per Node.js version: install Corepack again for each version you add.
:::

:::verify
```bash
node --version
python3 --version
docker compose version
```
Node.js prints `v22.18.0` or later, Python prints `Python 3.12`, and Compose prints its version. In your OpenCTI clone, `yarn --version` prints `4.18.1`.
:::

## Install the tools for OpenAEV

:::steps
1. Install Java 21 and Maven:

   ```bash
   sudo apt-get install -y openjdk-21-jdk maven
   ```

   Ubuntu 24.04 ships Maven 3.8. If `mvn -version` reports a version older than 3.9, install Maven from [maven.apache.org](https://maven.apache.org/download.cgi).

2. Install Node.js 24 and Corepack for it:

   ```bash
   nvm install 24
   nvm use 24
   npm install -g corepack
   corepack enable
   ```

   Node.js 24 also meets the OpenCTI requirement (22.18.0 or later), but OpenCTI is built and tested on Node.js 22. Switch with `nvm use 22` and `nvm use 24` when you move from one repository to the other.
:::

:::verify
```bash
java -version
mvn -version
node --version
```
Java and Maven report Java 21, Maven reports 3.9 or later, and Node.js prints `v24.11.0` or later.
:::

:::example[A new laptop for an OpenCTI fix]
A developer at Example CERT gets a fresh Ubuntu 24.04 laptop to fix how OpenCTI displays the reports on Velvet Harbor. They only need the OpenCTI tools, with Docker:

```bash
sudo apt-get update
sudo apt-get install -y git curl build-essential python3 python3-venv python3-dev docker.io docker-compose-v2
sudo usermod -aG docker "$USER"
echo 'vm.max_map_count=262144' | sudo tee /etc/sysctl.d/99-elasticsearch.conf
sudo sysctl --system
# after installing nvm and opening a new terminal
nvm install 22
nvm alias default 22
npm install -g corepack
```

After logging out and back in, `docker compose version` answers without `sudo`, and they continue with [Build and run from source](/develop/contribute/build-from-source/).
:::

## Choose an editor

The repositories carry run configurations for IntelliJ IDEA: `opencti-platform/opencti-dev/*.run.xml` for OpenCTI (**1 - Backend start**, **1 - Frontend start**, **Backend docker compose**), `openaev-dev/*.run.xml` for OpenAEV (**Backend start**, **Frontend start**). IntelliJ IDEA detects them when you open the repository root. Visual Studio Code works too, without these configurations.

## What's next

- [Build and run from source](/develop/contribute/build-from-source/): clone, configure, start the services and the platform, run the tests.
- [Prepare a Windows development environment](/develop/contribute/environment/windows/), for a Windows workstation.
- [Develop XTM Composer](/develop/contribute/xtm-composer/), for the Rust component that runs integrations.
