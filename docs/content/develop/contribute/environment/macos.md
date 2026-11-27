---
title: Prepare a macOS development environment
sidebar_label: Environment on macOS
description: Install on macOS, Apple silicon or Intel, the container engine, Node.js, Yarn, Python and Java versions that the OpenCTI and OpenAEV source code requires.
doc_type: how-to
products:
  - opencti
  - openaev
personas: [developer]
owner: docs-platform
last_reviewed: 2026-11-09
sidebar_position: 3
no_screenshot: "This page installs command-line tools on a workstation: there is no module screen to show."
migrated_from:
  - openaev:/development/environment-macos/
---

This page installs on macOS the tools you need to run OpenCTI or OpenAEV from source: a container engine for their services, and the exact Node.js, Yarn, Python and Java versions their code requires. It works on Apple silicon and on Intel Macs, with [Homebrew](https://brew.sh/). When the tools are in place, continue with [Build and run from source](/develop/contribute/build-from-source/).

::diagram{id=develop-contribute-build-from-source}

## Before you start

- A recent macOS, with administrator rights, and Homebrew installed.
- Enough memory to give the Linux virtual machine of the container engine 8 GB: the development services run in it, and Elasticsearch alone starts with a 2 GB Java heap in both modules.
- The versions below come from the code of OpenCTI 7.261030 and OpenAEV 3.261029. Contributions go to the default branches, `master` for OpenCTI and `main` for OpenAEV.

| Tool | OpenCTI | OpenAEV | Where the code sets it |
| --- | --- | --- | --- |
| Node.js | 22.18.0 or later; the images and CI run Node.js 22 | 24.11.0 or later | `engines` in `package.json` |
| Yarn | 4.18.1, through Corepack | 4.18.1, through Corepack | `packageManager` in `package.json` |
| Python | 3.12 | Not needed | Images `node:22-alpine3.23` and `python:3.12-alpine` |
| Java | Not needed | 21 | `java.version` in `pom.xml` |
| Maven | Not needed | 3.9 or later | OpenAEV development guide |
| Container engine | Docker Desktop or Colima, with Compose | Docker Desktop or Colima, with Compose | `opencti-dev` and `openaev-dev` |

## Install the shared tools

:::::steps
1. Install the Xcode Command Line Tools, which bring Git and the C and C++ compiler:

   ```bash
   xcode-select --install
   ```

   OpenCTI compiles two native Node.js modules at install time, `re2` and `node-calls-python`: they need the compiler.

2. Install a container engine with Compose. Pick one.

   ::::tabs
   :::tab[Docker Desktop]
   Install [Docker Desktop for Mac](https://docs.docker.com/desktop/setup/install/mac-install/), start it and wait until the engine runs. In **Settings** > **Resources**, give its virtual machine at least 8 GB of memory. Docker Desktop is free for personal use, education and small businesses; larger organizations need a paid subscription.
   :::
   :::tab[Colima]
   Install Colima, the Docker command line and Compose, then start a virtual machine large enough for Elasticsearch:

   ```bash
   brew install colima docker docker-compose
   colima start --cpu 4 --memory 8 --disk 40
   ```

   Homebrew installs Compose as a plugin of the Docker command line. Add its folder to `~/.docker/config.json` so that `docker compose` finds it:

   ```json
   {
     "cliPluginsExtraDirs": ["/opt/homebrew/lib/docker/cli-plugins"]
   }
   ```

   On an Intel Mac, the folder is `/usr/local/lib/docker/cli-plugins`. If `docker compose pull` then fails on `docker-credential-desktop`, remove the `"credsStore": "desktop"` line that Docker Desktop left in the same file.
   :::
   ::::

3. Check the memory map limit of the virtual machine, which Elasticsearch needs at 262144 or more:

   ```bash
   docker run --rm alpine cat /proc/sys/vm/max_map_count
   ```

   With Colima, raise it when it is lower, after each start of the virtual machine:

   ```bash
   colima ssh -- sudo sysctl -w vm.max_map_count=262144
   ```

4. Install nvm, the Node.js version manager, with the install script of [its README](https://github.com/nvm-sh/nvm/blob/master/README.md#installing-and-updating), then open a new terminal.
:::::

## Install the tools for OpenCTI

:::steps
1. Install Python 3.12:

   ```bash
   brew install python@3.12
   ```

   Create the virtual environment of [Build and run from source](/develop/contribute/build-from-source/) with this version, `python3.12 -m venv ../.python-venv`: the back end reads its packages from the `python3.12` folder of the environment.

2. Install Node.js 22 and make it the default:

   ```bash
   nvm install 22
   nvm alias default 22
   ```

3. Install Corepack, which fetches the Yarn version that `package.json` pins when you run `yarn` in the repository:

   ```bash
   npm install -g corepack
   ```

   nvm keeps global packages per Node.js version: install Corepack again for each version you add.
:::

:::verify
```bash
node --version
python3.12 --version
docker compose version
```
Node.js prints `v22.18.0` or later, Python prints `Python 3.12`, and Compose prints its version. In your OpenCTI clone, `yarn --version` prints `4.18.1`.
:::

The Elasticsearch service of `opencti-platform/opencti-dev/docker-compose.yml` sets `ES_JAVA_OPTS=-Xms2G -Xmx2G` and nothing for Apple silicon. If it stops at start on an Apple silicon Mac, add `-XX:+IgnoreUnrecognizedVMOptions -XX:UseSVE=0` to `ES_JAVA_OPTS` in a Compose override file of your own, as OpenAEV's development stack does.

## Install the tools for OpenAEV

:::steps
1. Install Java 21 and Maven:

   ```bash
   brew install openjdk@21 maven
   ```

2. `openjdk@21` is not linked into your path by Homebrew. Add it to `~/.zshrc`, then open a new terminal:

   ```bash
   export JAVA_HOME="$(brew --prefix openjdk@21)"
   export PATH="$JAVA_HOME/bin:$PATH"
   ```

   Without it, `java -version` answers `Unable to locate a Java Runtime`.

3. Install Node.js 24 and Corepack for it:

   ```bash
   nvm install 24
   nvm use 24
   npm install -g corepack
   corepack enable
   ```

   Node.js 24 also meets the OpenCTI requirement, but OpenCTI is built and tested on Node.js 22. Switch with `nvm use 22` and `nvm use 24` when you move from one repository to the other.
:::

:::verify
```bash
java -version
mvn -version
node --version
```
Java and Maven report Java 21, Maven reports 3.9 or later, and Node.js prints `v24.11.0` or later.
:::

The Elasticsearch and OpenSearch services of `openaev-dev/docker-compose.yml` already pass `-XX:UseSVE=0` for Apple silicon: you change nothing there.

:::example[A MacBook for an OpenAEV fix]
A developer at Example CERT gets a MacBook with Apple silicon to fix how OpenAEV shows the results of the Velvet Harbor simulation on the front desks of Grand Meridian Hotels. They only need the OpenAEV tools, with Colima:

```bash
xcode-select --install
brew install colima docker docker-compose openjdk@21 maven
colima start --cpu 4 --memory 8 --disk 40
colima ssh -- sudo sysctl -w vm.max_map_count=262144
echo 'export JAVA_HOME="$(brew --prefix openjdk@21)"' >> ~/.zshrc
echo 'export PATH="$JAVA_HOME/bin:$PATH"' >> ~/.zshrc
# after installing nvm and opening a new terminal
nvm install 24
npm install -g corepack
corepack enable
```

`java -version` reports Java 21 and `docker compose version` answers, and they continue with [Build and run from source](/develop/contribute/build-from-source/).
:::

## Choose an editor

The repositories carry run configurations for IntelliJ IDEA: `opencti-platform/opencti-dev/*.run.xml` for OpenCTI, `openaev-dev/*.run.xml` for OpenAEV. IntelliJ IDEA detects them when you open the repository root. Visual Studio Code works too, without these configurations.

## What's next

- [Build and run from source](/develop/contribute/build-from-source/): clone, configure, start the services and the platform, run the tests.
- [Prepare an Ubuntu development environment](/develop/contribute/environment/ubuntu/), for a Linux workstation.
- [Prepare a Windows development environment](/develop/contribute/environment/windows/), for a Windows workstation.
