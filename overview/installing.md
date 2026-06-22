---
aliases: ["/overview/installing/"]
lastmod: 2019-03-11
date: 2016-07-01
menu:
  community_current:
    parent: "000 Getting Started"
linktitle: Installation
title: Pucora API Gateway Installation Guide
description: Follow our step-by-step installation guide to set up and configure Pucora API Gateway, enabling efficient and scalable API management.
weight: 10
---
Pucora is a **single binary file** that does not require any external libraries to work. To install Pucora choose your operative system in the downloads section or use the Docker image.

{{< button-group >}}
{{< button url="/download/" text="Download" >}}<svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 12a9 9 0 01-9 9m9-9a9 9 0 00-9-9m9 9H3m9 9a9 9 0 01-9-9m9 9c1.657 0 3-4.03 3-9s-1.343-9-3-9m0 18c-1.657 0-3-4.03-3-9s1.343-9 3-9m-9 9a9 9 0 019-9" />
</svg>{{< /button >}}
{{< button url="https://designer.pucora.in/" type="inversed" >}}Generate a config file{{< /button >}}
{{< /button-group >}}

## Installation
You can install (or just run) Pucora using different options.
{{< note title="Just exploring?" type="question" >}}
Use the [Pucora Playground](https://github.com/pucora/playground-community) if you want to play with Pucora without configuring it. The Playground comes with several flavors of Pucora and a mock API. Everything is ready to start playing, just do a `docker compose up`!
{{< /note >}}

### Docker
The easiest way to get started is by pulling and running the [Pucora image](https://hub.docker.com/_/pucora) from the Docker Hub.
{{< terminal title="Running Pucora using the Docker container" >}}
docker run -p 8080:8080 -v $PWD:/etc/pucora/ {{< product image >}} run --config /etc/pucora/pucora.json
{{< /terminal >}}

If you choose not to mount the volume (the `-v`), a default `pucora.json` serving a `/__health` endpoint will be used. The volume expects to find a `pucora.json` in the current directory ([generate your first here](https://designer.pucora.in/)).

### AWS and Azure VM
See the [downloads page](/download/) for pre-built virtual machines in Azure and AWS.

### Mac OS X
The [Homebrew](https://brew.sh/) formula will download the source code, build the binary, and link the binary for you. The installation might take a while.

{{< terminal title="Install on Mac via Brew" >}}
brew install pucora
{{< /terminal >}}

After the installation completes go to [Using Pucora](/docs/overview/run/)

### Linux

#### CentOS, Oracle Linux, and Redhat (yum)
The installation process requires following these steps:

1. Install the repo package
2. Install the Pucora package
3. Start the Pucora service

Paste this in the terminal:
{{< terminal title="Yum based" >}}
rpm -Uvh {{< product download_repo >}}/rpm/{{< product rpm_source >}}
yum install -y pucora
systemctl start pucora
{{< /terminal >}}

#### Fedora
Paste this in the terminal:
{{< terminal title="DNF based" >}}
rpm -Uvh {{< product download_repo >}}/rpm/{{< product rpm_source >}}
dnf install -y pucora
systemctl start pucora
{{< /terminal >}}

The current Pucora version will run at least in Centos 7 and Fedora 24

#### Debian and Ubuntu

The installation process requires following these steps:

1. Add the key
2. Add the repo to the sources.list
3. Update your package list
4. Install the Pucora service

Bottom line:
{{< terminal title="DEB based" >}}
apt install -y ca-certificates gnupg
apt-key adv --keyserver keyserver.ubuntu.com --recv {{< param pgp_key >}}
echo "deb {{< product download_repo >}}/apt stable main" | tee /etc/apt/sources.list.d/pucora.list
apt-get update
apt-get install -y pucora
{{< /terminal >}}

Minimum supported versions are Debian 8, and Ubuntu 16.x.

#### Generic Linux (`tar.gz`)
You can also [download](/download/) the `tar.gz` and decompress it anywhere. Instructions to check the SHA and PGP signature [here](/docs/overview/verifying-packages/).

## Compile from source
As Pucora is open source you can opt for [building the binary](https://github.com/pucora/pucora-ce). The binary you will produce is the same you can get in our download page, only that compiling it yourself always feels good!
