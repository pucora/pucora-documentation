---
aliases: ["/overview/installing/"]
lastmod: 2019-03-11
date: 2016-07-01
menu:
  community_current:
    parent: "000 Getting Started"
linktitle: Installation
title: Velonetics API Gateway Installation Guide
description: Follow our step-by-step installation guide to set up and configure Velonetics API Gateway, enabling efficient and scalable API management.
weight: 10
---
Velonetics is a **single binary file** that does not require any external libraries to work. To install Velonetics choose your operative system in the downloads section or use the Docker image.

{{< button-group >}}
{{< button url="/download/" text="Download" >}}<svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 12a9 9 0 01-9 9m9-9a9 9 0 00-9-9m9 9H3m9 9a9 9 0 01-9-9m9 9c1.657 0 3-4.03 3-9s-1.343-9-3-9m0 18c-1.657 0-3-4.03-3-9s1.343-9 3-9m-9 9a9 9 0 019-9" />
</svg>{{< /button >}}
{{< button url="https://designer.velonetics.io/" type="inversed" >}}Generate a config file{{< /button >}}
{{< /button-group >}}

## Installation
You can install (or just run) Velonetics using different options.
{{< note title="Just exploring?" type="question" >}}
Use the [Velonetics Playground](https://github.com/velonetics/playground-community) if you want to play with Velonetics without configuring it. The Playground comes with several flavors of Velonetics and a mock API. Everything is ready to start playing, just do a `docker compose up`!
{{< /note >}}

### Docker
The easiest way to get started is by pulling and running the [Velonetics image](https://hub.docker.com/_/velonetics) from the Docker Hub.
{{< terminal title="Running Velonetics using the Docker container" >}}
docker run -p 8080:8080 -v $PWD:/etc/velonetics/ {{< product image >}} run --config /etc/velonetics/velonetics.json
{{< /terminal >}}

If you choose not to mount the volume (the `-v`), a default `velonetics.json` serving a `/__health` endpoint will be used. The volume expects to find a `velonetics.json` in the current directory ([generate your first here](https://designer.velonetics.io/)).

### AWS and Azure VM
See the [downloads page](/download/) for pre-built virtual machines in Azure and AWS.

### Mac OS X
The [Homebrew](https://brew.sh/) formula will download the source code, build the binary, and link the binary for you. The installation might take a while.

{{< terminal title="Install on Mac via Brew" >}}
brew install velonetics
{{< /terminal >}}

After the installation completes go to [Using Velonetics](/docs/overview/run/)

### Linux

#### CentOS, Oracle Linux, and Redhat (yum)
The installation process requires following these steps:

1. Install the repo package
2. Install the Velonetics package
3. Start the Velonetics service

Paste this in the terminal:
{{< terminal title="Yum based" >}}
rpm -Uvh {{< product download_repo >}}/rpm/{{< product rpm_source >}}
yum install -y velonetics
systemctl start velonetics
{{< /terminal >}}

#### Fedora
Paste this in the terminal:
{{< terminal title="DNF based" >}}
rpm -Uvh {{< product download_repo >}}/rpm/{{< product rpm_source >}}
dnf install -y velonetics
systemctl start velonetics
{{< /terminal >}}

The current Velonetics version will run at least in Centos 7 and Fedora 24

#### Debian and Ubuntu

The installation process requires following these steps:

1. Add the key
2. Add the repo to the sources.list
3. Update your package list
4. Install the Velonetics service

Bottom line:
{{< terminal title="DEB based" >}}
apt install -y ca-certificates gnupg
apt-key adv --keyserver keyserver.ubuntu.com --recv {{< param pgp_key >}}
echo "deb {{< product download_repo >}}/apt stable main" | tee /etc/apt/sources.list.d/velonetics.list
apt-get update
apt-get install -y velonetics
{{< /terminal >}}

Minimum supported versions are Debian 8, and Ubuntu 16.x.

#### Generic Linux (`tar.gz`)
You can also [download](/download/) the `tar.gz` and decompress it anywhere. Instructions to check the SHA and PGP signature [here](/docs/overview/verifying-packages/).

## Compile from source
As Velonetics is open source you can opt for [building the binary](https://github.com/velonetics/velonetics-ce). The binary you will produce is the same you can get in our download page, only that compiling it yourself always feels good!
