---
title: Prepare a Windows development environment
sidebar_label: Environment on Windows
description: Set up Windows to run OpenCTI from source in WSL 2, and OpenAEV natively or in WSL 2, with the tool versions their code requires.
doc_type: how-to
products:
  - opencti
  - openaev
personas: [developer]
owner: docs-platform
last_reviewed: 2026-11-01
sidebar_position: 2
no_screenshot: "This page installs command-line tools on a workstation: there is no module screen to show."
migrated_from:
  - opencti:/development/environment_windows/
  - openaev:/development/environment-windows/
---

This page prepares a Windows workstation to run OpenCTI or OpenAEV from source. For OpenCTI, you work in Windows Subsystem for Linux (WSL) 2 with Ubuntu, because its development scripts expect a Linux shell. OpenAEV runs natively on Windows or in WSL 2. When the tools are in place, continue with [Build and run from source](/develop/contribute/build-from-source/).

::diagram{id=develop-contribute-build-from-source}

## Before you start

- Windows 10 version 2004 or later, or Windows 11, with administrator rights.
- At least 8 GB of memory for OpenCTI: its development Elasticsearch starts with a 2 GB Java heap.
- Docker Desktop is free for personal use, education and small businesses; larger organizations need a paid subscription. Check its license terms for your organization, or run Docker Engine inside WSL 2 instead.

## Why OpenCTI needs WSL 2

The OpenCTI code assumes Linux in three places:

- `scripts/with_python_virtual_env.mjs`, behind the `deps:venv`, `dev:venv` and `test:venv` scripts, runs `sh` and sources `<venv>/bin/activate`. A Windows virtual environment has `Scripts\activate` instead.
- The native modules `re2` and `node-calls-python` compile at install time. On Windows, they would need Visual Studio Build Tools with the **Desktop development with C++** workload, and Python headers.
- The release images and the CI build on Linux (`node:22-alpine3.23`), so a Linux environment behaves like them.

The repository forces LF line endings (`* text=auto eol=lf` in `.gitattributes`), so a clone works the same on both sides.

## Set up WSL 2 for OpenCTI

:::::steps
1. In PowerShell run as administrator, install WSL 2 with Ubuntu 24.04, then restart Windows:

   ```powershell
   wsl --install -d Ubuntu-24.04
   ```

   At the first start, Ubuntu asks for a user name and a password for your Linux account.

2. Raise the memory map limit that Elasticsearch needs. Create or edit `%USERPROFILE%\.wslconfig` with:

   ```ini
   [wsl2]
   kernelCommandLine = "sysctl.vm.max_map_count=262144"
   ```

   Then restart WSL from PowerShell with `wsl --shutdown`. The setting applies to every WSL 2 distribution, Docker Desktop's included.

3. Install a container engine. Pick one.

   ::::tabs
   :::tab[Docker Desktop]
   Install [Docker Desktop for Windows](https://docs.docker.com/desktop/setup/install/windows-install/) with the WSL 2 backend. In Docker Desktop, open **Settings** > **Resources** > **WSL integration** and turn on **Ubuntu-24.04**. The `docker` command then works inside Ubuntu.
   :::
   :::tab[Docker Engine in WSL 2]
   Install Docker Engine inside Ubuntu, as [Prepare an Ubuntu development environment](/develop/contribute/environment/ubuntu/#install-the-shared-tools) describes. Skip its `vm.max_map_count` step: `.wslconfig` sets it.
   :::
   ::::

4. Open the Ubuntu terminal and install the rest of the tools as [Prepare an Ubuntu development environment](/develop/contribute/environment/ubuntu/) describes: Git, `build-essential`, Python 3.12 with `python3-venv` and `python3-dev`, nvm, Node.js 22 and Corepack.

5. Clone the repository inside the Linux file system, in your Linux home folder, not under `/mnt/c`. The file watchers of the front end and the back end are slow and can miss changes on the Windows file system.
:::::

:::verify
In the Ubuntu terminal:

```bash
node --version
python3 --version
docker compose version
cat /proc/sys/vm/max_map_count
```
Node.js prints `v22.18.0` or later, Python prints `Python 3.12`, Compose prints its version, and the last command prints `262144`.
:::

A browser on Windows reaches the servers you start in WSL 2 at `http://localhost:<port>`. Visual Studio Code with its WSL extension, or IntelliJ IDEA with its WSL support, opens a project stored in WSL 2.

## Set up Windows for OpenAEV

OpenAEV builds with Maven and Node.js, which run natively on Windows. The repository gives Windows variants of its setup commands, such as `setup-auto-db.ps1`. You can also follow the Ubuntu page inside WSL 2.

:::steps
1. Install Java 21, Maven 3.9 or later and Git:

   ```powershell
   winget install EclipseAdoptium.Temurin.21.JDK
   winget install Apache.Maven
   winget install Git.Git
   ```

2. Install [Docker Desktop for Windows](https://docs.docker.com/desktop/setup/install/windows-install/) with the WSL 2 backend. Elasticsearch needs the `.wslconfig` setting of [Set up WSL 2 for OpenCTI](#set-up-wsl-2-for-opencti).

3. Install [nvm-windows](https://github.com/coreybutler/nvm-windows/releases), then Node.js 24 and Corepack:

   ```powershell
   nvm install 24
   nvm use 24
   npm install -g corepack
   corepack enable
   ```

4. Run the commands of [Build and run from source](/develop/contribute/build-from-source/) in Git Bash, installed with Git for Windows: they use bash syntax.
:::

:::verify
```powershell
java -version
mvn -version
node --version
docker compose version
```
Java and Maven report Java 21, Maven reports 3.9 or later, Node.js prints `v24.11.0` or later, and Compose prints its version.
:::

:::example[A Windows laptop for OpenCTI work]
A developer at Example CERT works on a Windows 11 laptop with Docker Desktop. They want to write a connector that imports a hotel-sector feed and test it against OpenCTI built from source:

```powershell
wsl --install -d Ubuntu-24.04
# after the restart, in %USERPROFILE%\.wslconfig:
#   [wsl2]
#   kernelCommandLine = "sysctl.vm.max_map_count=262144"
wsl --shutdown
```

They turn on **Ubuntu-24.04** in **Settings** > **Resources** > **WSL integration** of Docker Desktop, install the OpenCTI tools in the Ubuntu terminal, and clone OpenCTI into `~/opencti`. The platform they start there answers at `http://localhost:3000` in their Windows browser.
:::

## What's next

- [Build and run from source](/develop/contribute/build-from-source/): clone, configure, start the services and the platform, run the tests.
- [Prepare an Ubuntu development environment](/develop/contribute/environment/ubuntu/), the commands to run inside WSL 2.
- [Develop XTM Composer](/develop/contribute/xtm-composer/), for the Rust component that runs integrations.
