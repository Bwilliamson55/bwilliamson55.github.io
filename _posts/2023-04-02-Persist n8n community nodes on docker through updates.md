---
title: Persist n8n community nodes on docker through updates
date: 2023-04-02 16:00
ategories: [ETL, n8n]
author: bwilliamson
tags: [n8n, etl, n8n-nodes]
---

# TLDR

[n8n](https://n8n.io) via an [App service](https://www.digitalocean.com/products/app-platform), using a Docker image, will lose community nodes every rebuild. We can "persist" these custom packages by using a Dockerfile as our App service context rather than the docker image itself.

There is other ways to achieve this, which I'll touch on, but this is what I've implemented and therefore will share.


<details markdown="1">
  <summary>Copy the current App Spec from here</summary>

Private information is redacted of course.

Like many hosts, you can do your infra "as code".
You could almost copy paste this 1:1 into a new app on Digital Ocean and be done.

This is much more than necessary though- the key bits are at the bottom. The ***github key, the source_dir, and the dockerfile_path*** are all that's needed to be honest.

See granular docs here: [Digital Ocean App spec docs](https://docs.digitalocean.com/products/app-platform/reference/app-spec/)

```yaml
databases:
- engine: PG
  name: db
  num_nodes: 1
  size: basic-xs
  version: "12"
envs:
- key: N8N_BASIC_AUTH_ACTIVE
  scope: RUN_AND_BUILD_TIME
  value: "false"
- key: N8N_PORT
  scope: RUN_AND_BUILD_TIME
  value: "5678"
- key: EXECUTIONS_PROCESS
  scope: RUN_AND_BUILD_TIME
  value: main
- key: N8N_SKIP_WEBHOOK_DEREGISTRATION_SHUTDOWN
  scope: RUN_AND_BUILD_TIME
  value: "True"
name: my-super-special-app
region: nyc
services:
- cors:
    allow_headers:
    - '*'
    allow_methods:
    - GET
    - HEAD
    - POST
    - DELETE
    - PATCH
    - PUT
    - CONNECT
    - OPTIONS
    - TRACE
    allow_origins:
    - prefix: https://ondigitalocean.app
    - prefix: https://codepen.io
    - prefix: https://cdpn.io
  envs:
  - key: DB_TYPE
    scope: RUN_AND_BUILD_TIME
    value: postgresdb
  - key: DB_POSTGRESDB_HOST
    scope: RUN_AND_BUILD_TIME
    value: app-lots-of-words-here-get-this-from-db-info
  - key: DB_POSTGRESDB_PORT
    scope: RUN_AND_BUILD_TIME
    value: "25060"
  - key: DB_POSTGRESDB_DATABASE
    scope: RUN_AND_BUILD_TIME
    value: db
  - key: DB_POSTGRESDB_USER
    scope: RUN_AND_BUILD_TIME
    value: db
  - key: DB_POSTGRESDB_PASSWORD
    scope: RUN_AND_BUILD_TIME
    value: 1234-super-secret-from-db-info
  - key: DB_POSTGRESDB_SSL_REJECT_UNAUTHORIZED
    scope: RUN_AND_BUILD_TIME
    value: "False"
  - key: N8N_HOST
    scope: RUN_AND_BUILD_TIME
    value: my-app-name.ondigitalocean.app/
  - key: WEBHOOK_URL
    scope: RUN_AND_BUILD_TIME
    value: https://my-app-name.ondigitalocean.app
  - key: N8N_ENCRYPTION_KEY
    scope: RUN_AND_BUILD_TIME
    value: B+abcDefgHijkLMnOpQrstuvwxyz-this-is-an-example
  - key: N8N_PROTOCOL
    scope: RUN_AND_BUILD_TIME
    value: https
  - key: N8N_PUSH_BACKEND
    scope: RUN_AND_BUILD_TIME
    value: websocket
  - key: NODE_FUNCTION_ALLOW_BUILTIN
    scope: RUN_AND_BUILD_TIME
    value: '*'
  - key: NODE_FUNCTION_ALLOW_EXTERNAL
    scope: RUN_AND_BUILD_TIME
    value: '*'
  github:
    branch: master
    deploy_on_push: true
    repo: Bwilliamson55/n8n-custom-images
  health_check:
    http_path: /
    initial_delay_seconds: 60
    period_seconds: 15
    timeout_seconds: 5
  dockerfile_path: browserless_clickuplookup/Dockerfile
  http_port: 5678
  instance_count: 1
  instance_size_slug: basic-xs
  name: n-8-nio-n-8-n
  routes:
  - path: /
  source_dir: browserless_clickuplookup

```
</details>

---

# Ephemeral

Cool word right? I think so. But this is the problem we're solving - Docker containers intrinsically will be ephemeral and any changes we make to their file system will be lost when the container rebuilds, redeploys, or updates. Anything that pulls a new image, or rebuilds the container, will wipe the files that container was holding. This is a good thing, but complicates certain features of applications we have Dockerized.

The focus here is n8n - the image for this can be pulled with `n8nio/n8n:latest`
One of the features in n8n is installing custom nodes built by the community with just a few clicks. This is really nice but does not persist in a containerized environment. This is bad because should our container crash and rebuild, any workflows depending on those custom nodes will fail to work until we reinstall the missing nodes.

Community nodes are just npm packages though- so they can be installed with a simple `npm install n8n-nodes-nodeName` command. This is required of community nodes- that they be published on npm. This makes this problem an easy solve. Well, depending on your hosting situation.

---

# Hosting

For most of my work I use [Digital Ocean](https://www.digitalocean.com), and in this case I'm outlining a few ways we can use a [Digital Ocean App](https://www.digitalocean.com/products/app-platform) with Docker images and files. This of course is not the only way to [host n8n](https://docs.n8n.io/choose-n8n/).

- [n8n cloud](https://n8n.io/cloud/)
- [n8n desktop app](https://docs.n8n.io/choose-n8n/desktop-app/)
- [n8n CLI install](https://docs.n8n.io/hosting/installation/npm/)
- [n8n Docker images](https://docs.n8n.io/hosting/installation/docker/)

Are just the big categories of ways to host this that come to mind. Docker images especially can be hosted in a multitude of ways depending on the provider or platform. I prefer Digital Ocean so for other providers like the big three, you'll need to adapt these instructions to your needs. n8n provides fantastic guides you can refer to above.

Using the [Digital Ocean App](https://www.digitalocean.com/products/app-platform) platform to run n8n has been a breeze! You can spin up a new n8n instance with postgres in less than an hour. Easily!

---

# The Dockerfile

[The sample Dockerfile(s)](https://github.com/n8n-io/n8n/tree/master/docker/images) from n8n are what we'll be using. Specifically [this one](https://github.com/n8n-io/n8n/blob/master/docker/images/n8n/Dockerfile).

```Dockerfile
ARG NODE_VERSION=16
FROM n8nio/base:${NODE_VERSION}

ARG N8N_VERSION
RUN if [ -z "$N8N_VERSION" ] ; then echo "The N8N_VERSION argument is missing!" ; exit 1; fi

ENV N8N_VERSION=${N8N_VERSION}
ENV NODE_ENV=production
RUN set -eux; \
	apkArch="$(apk --print-arch)"; \
	case "$apkArch" in \
	'armv7') apk --no-cache add --virtual build-dependencies python3 build-base;; \
	esac && \
	npm install -g --omit=dev n8n@${N8N_VERSION} && \
	case "$apkArch" in \
	'armv7') apk del build-dependencies;; \
	esac && \
	find /usr/local/lib/node_modules/n8n -type f -name "*.ts" -o -name "*.js.map" -o -name "*.vue" | xargs rm && \
	rm -rf /root/.npm

# Set a custom user to not have n8n run as root
USER root
WORKDIR /data
RUN apk --no-cache add su-exec
COPY docker-entrypoint.sh /docker-entrypoint.sh
ENTRYPOINT ["tini", "--", "/docker-entrypoint.sh"]
```

We'll add a few npm packages to the file, and save it along with the `docker-entrypoint.sh`:

```Dockerfile
#Dockerfile
ARG NODE_VERSION=18
FROM n8nio/base:${NODE_VERSION}

ARG N8N_VERSION

ENV N8N_VERSION=latest
ENV NODE_ENV=production
RUN set -eux; \
	apkArch="$(apk --print-arch)"; \
	case "$apkArch" in \
	'armv7') apk --no-cache add --virtual build-dependencies python3 build-base;; \
	esac && \
	npm install -g --omit=dev n8n@latest && \
	case "$apkArch" in \
	'armv7') apk del build-dependencies;; \
	esac && \
	find /usr/local/lib/node_modules/n8n -type f -name "*.ts" -o -name "*.js.map" -o -name "*.vue" | xargs rm && \
	rm -rf /root/.npm

################# Added this stuff
# Install the browserless and clickuplookup package
RUN cd /usr/local/lib/node_modules/n8n && npm install n8n-nodes-browserless && npm install n8n-nodes-clickuplookup
################

# Set a custom user to not have n8n run as root
USER root
WORKDIR /data
RUN apk --no-cache add su-exec
COPY docker-entrypoint.sh /docker-entrypoint.sh
RUN chmod +x /docker-entrypoint.sh
ENTRYPOINT ["tini", "--", "/docker-entrypoint.sh"]
```

d**ocker-entrypoint.sh:**

```bash
#!/bin/sh

if [ -d /root/.n8n ] ; then
  chmod o+rx /root
  chown -R node /root/.n8n
  ln -s /root/.n8n /home/node/
fi

chown -R node /home/node

if [ "$#" -gt 0 ]; then
  # Got started with arguments
  exec su-exec node "$@"
else
  # Got started without arguments
  exec su-exec node n8n
fi
```
Upload this to your repository in github, and we're half done!
To see this exact example, [see it on github](https://github.com/Bwilliamson55/n8n-custom-images/tree/master/browserless_clickuplookup).

Don't worry- the only thing here we've changed from the official version is that extra `RUN` line. You can copy paste newer versions, most likely, [directly from the n8n repositories examples.](https://github.com/n8n-io/n8n/tree/master/docker/images/n8n)

# For an existing App

Go into your app spec, which can be found under your app's "Settings" tab, and replace this part:

![docker hub image spec example](/assets/img/post%20images/etl/n8n/20230403/01-docker_hub_image_spec.png){: .normal }

with this:

![dockerfile spec example](/assets/img/post%20images/etl/n8n/20230403/02-dockerfile_spec.png){ .normal }

Save. Done.
Your app will now re-deploy and build the image fresh. Each rebuild will get the newest version without losing the custom nodes.

# For a new App

- Create a project
- Add an app resource
  - ![Create app resource](/assets/img/post%20images/etl/n8n/20230403/03-create-app.png){: .normal }
- Select github repository as the source - pointing to the dockerfile location
  - ![Select your github repository](/assets/img/post%20images/etl/n8n/20230403/04-target-dockerfile-location.png){: .normal }
  - This GUI requires you link your github account! Feel free to fork the above repository for your needs.
- Update plan and other details
  - Update the app's http port ***from 8080 to 5678***
    - ![Edit app settings for port](/assets/img/post%20images/etl/n8n/20230403/05-edit-app-port.png){: .normal }
    - ![Edit app port two](/assets/img/post%20images/etl/n8n/20230403/05-edit-app-port-2.png){: .normal }
  - Edit plan to change pricing
    - ![Edit app plan](/assets/img/post%20images/etl/n8n/20230403/05-edit-plan-1.png){: .normal }
    - ![Edit app plan details](/assets/img/post%20images/etl/n8n/20230403/06-edit-plan-2.png){: .normal }
- Add a DB
  - ![Add database](/assets/img/post%20images/etl/n8n/20230403/07-add-db.png){: .normal }
- Create resources - expect the first deploy to fail, we still have a few things to change.
  - ![Create Resources view](/assets/img/post%20images/etl/n8n/20230403/07-review-create.png){: .normal }

## Configuration

Under the app settings tab, even while it's deploying, we can update the environment variables.

Going into the DB's settings tab, we can get the required connection information once it's deployed:

![Db settings tab](/assets/img/post%20images/etl/n8n/20230403/08-db-settings.png){: .normal }

![Db connection details](/assets/img/post%20images/etl/n8n/20230403/09-db-connection-details.png){: .normal }

Under the app's settings- click "edit" near "Environment Variables", and then click "bulk edit":

![Edit app env vars](/assets/img/post%20images/etl/n8n/20230403/10-env-vars.png){: .normal }

![Env vars bulk editor](/assets/img/post%20images/etl/n8n/20230403/11-env-vars.png){: .normal }

Edit these so your variables meet the connection details of your database, and the public URL of your app once it deploys the first time:

```config
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=app-my-host-name.com
DB_POSTGRESDB_PORT=25060
DB_POSTGRESDB_DATABASE=db
DB_POSTGRESDB_USER=db
DB_POSTGRESDB_PASSWORD=superSecretPassword
DB_POSTGRESDB_SSL_REJECT_UNAUTHORIZED=False
N8N_HOST=thePublicUrlOfTheAppAfterDeploy
WEBHOOK_URL=thePublicUrlOfTheAppAfterDeploy
N8N_ENCRYPTION_KEY=Long+key_string
N8N_PROTOCOL=https
N8N_PUSH_BACKEND=websocket
NODE_FUNCTION_ALLOW_BUILTIN=*
NODE_FUNCTION_ALLOW_EXTERNAL=*
N8N_BASIC_AUTH_ACTIVE=false
N8N_PORT=5678
EXECUTIONS_PROCESS=main
N8N_SKIP_WEBHOOK_DEREGISTRATION_SHUTDOWN=True
```

# Fin

That's all for this one, I hope you found this interesting!
