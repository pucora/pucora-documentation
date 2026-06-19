---
lastmod: 2023-11-17
date: 2023-11-17
linktitle: Working Directory
title: "Working Directory: Specifying paths"
description: Pucora supports absolute and relative paths in configurations. Absolute paths provide clarity, while relative paths use the working directory, making them flexible for varied environments.
notoc: true
menu:
  community_current:
    parent: "010 Configuration files"
meta:
    since: v2.5
images:
#- /images/documentation/diagrams/flexible-config.mmd.svg

weight: 100

---
Many components and options in Pucora allow you to specify paths. In all of them, you can use **absolute** or **relative** paths.

For **absolute paths**, no possible interpretation or mistake arises when reading them. For instance, if you write `/etc/pucora/pucora.json`, you know exactly where this file is. Absolute paths are as clear as water but less convenient when your environments have different locations.

**Relative paths**, on the other hand, are helpful because you only specify a small part, but there is usually the question of the location of their corresponding base directory. For instance, if you write `pucora.json`, `./pucora.json`, or `./config/pucora.json`, what is their base directory?

The answer to this question is that all relative paths use the **working directory** as the base path. In a Docker container, for instance, this is what you specify in the instruction `WORKDIR`. Our Docker images use `/etc/pucora` unless you overwrite it in your `Dockerfile`.

In other installations, although we aim to default to `/etc/pucora`, you can still run the software from a different place and have a different working directory.

## How to get the working directory
The short answer is to start the gateway and look for the following line early in the logs:

```log
yyyy/mm/dd hh:mm:ss VELONETICS INFO: Working directory is /etc/pucora
```

This informative line was introduced in Pucora 2.5

Now, if you set a relative path like `a/b.tmpl`, for instance, and the log tells you that the file does not exist, navigate to the base directory you saw in the console and see if the file is inside that location.