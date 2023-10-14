---
title: Breaking changes from n8n v1.0+
date: 2023-10-14 08:00
categories: [ETL, n8n]
author: bwilliamson
tags: [update, n8n, docker, npm]
---
If you're like me and use n8nio/n8n:latest in a dockerfile - you probably know this already.

# I broke it

I was lazy, and didn't update my primary n8n.io instance until this morning.
To all those that may use my repository, sorry! I hope you figured it out.

# I fixed it

TLDR is that n8n now runs as the `node` user, instead of `root`, which is a good thing for a lot of reasons. It's a bad thing for my Dockerfiles, which assumed differently. This has been corrected in my [custom-images repository](https://github.com/bwilliamson55/n8n-custom-images). Which, given the method the images take, 'custom' is kind of a stretch.

# What changed?

Many things are better now, check out the official migration guide here: [https://docs.n8n.io/1-0-migration-checklist/](https://docs.n8n.io/1-0-migration-checklist/)

What bit me was [this simple change](https://docs.n8n.io/1-0-migration-checklist/#permissions-change):
>When using Docker-based deployments, the n8n process is now run by the user node instead of root. This change increases security.
>
>If permission errors appear in your n8n container logs when starting n8n, you may need to update the permissions by executing the following command on the Docker host:
>.....

And I'm sure due to this, [the directories for custom nodes moved around a bit](https://docs.n8n.io/1-0-migration-checklist/#directory-for-installing-custom-nodes):
>n8n will no longer load custom nodes from its global node_modules directory. Instead, you must install (or link) them to ~/.n8n/custom (or a directory defined by CUSTOM_EXTENSION_ENV). Custom nodes that are npm packages will be located in ***~/.n8n/nodes***. If you have custom nodes that were linked using npm link into the global node_modules directory, you need to link them again, into ***~/.n8n/nodes*** instead.

#### Resulting simple dockerfile to include a custom node on startup and rebuild:
```dockerfile
FROM n8nio/n8n:latest
RUN mkdir ~/.n8n/nodes && cd ~/.n8n/nodes && npm install n8n-nodes-<node-name>
```
## Since we're here..

What else changed?

### [Execution order](https://docs.n8n.io/1-0-migration-checklist/#execution-order)
The branching execution order is now a bit more logical- each branch executes top to bottom completely before moving to the next branch.

### [Data retention defaults](https://docs.n8n.io/1-0-migration-checklist/#execution-data-retention)
> Starting from n8n 1.0, all successful, failed, and manual workflow executions will be saved by default.

> ... the EXECUTIONS_DATA_PRUNE setting will be ***enabled by default***, with EXECUTIONS_DATA_PRUNE_MAX_COUNT set to 10,000

Good!

### [Python support  ](https://docs.n8n.io/1-0-migration-checklist/#python-support-in-the-code-node)
I know a lot of people were looking forward to this. Release the pandas!

There's more, but these are impactful to me.


---
That's all for now, hope you found that interesting!
