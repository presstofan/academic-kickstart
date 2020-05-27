---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Scalable Shiny App Deployment with ShinyProxy, Traefik and Docker Swarm"
subtitle: "A setup tutorial for deploying ShinyProxy powered R Shiny apps in Docker Swarm mode with Traefik "
summary: "This post provides a guide to scale your ShinyProxy deployment of shiny apps with Docker Swarm and Traefik"
authors: [Yihui Fan]
tags: [R, R Shiny, ShinyProxy]
categories: [R Shiny]
date: 2020-05-26T20:51:35+01:00
lastmod: 2020-05-26T20:51:35+01:00
featured: false
draft: true

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

{{% toc %}}

## Introduction

If you search for R Shiny apps deployment and land on this post, chances are you are like me - a data scientist who just want to build a decent app to host a dashboard or a prediction model without going down to the rabbit hole of DevOps or the frontend / backend development. Don't worry, this is just the thing you need and I promise nothing below is too complicated. You can probably go through the following tutorial in half an hour. Let's start.

In this post, we will build a scalable, production-grade Shiny app powered by ShinyProxy and Docker Swarm and use Traefik to handle the SSL certificate (which gives you the little padlock before your domain name) , reverse proxy (i.e. route traffic from the 80 and 443 ports to your Shiny app and other places if needed) and load balancing. If you are not familiar with ShinyProxy and wonder why you should use it over the other options, I have written another post titled [Deploying R Shiny apps using ShinyProxy on Windows 10]({{< ref "/post/deploy-r-app-with-shinyproxy/index.md" >}}) that explains it in more detail. If you haven't used Docker technology before, it is also a good idea to check out that post first. In the interest of space, I won't cover too much details regarding Docker in this tutorial.

INSERT ARCHITECTURE GRAPH HERE

### Docker Swarm vs standard Docker containers

You probably wonder why you need Docker Swarm rather than just using docker-compose to deploy containers, particularly if you only have one server/node/instance. Docker Captain Bret Fisher explained it well [here](https://github.com/BretFisher/ama/issues/8). I summarised a couple of key benefits of Docker Swarm below:

* Docker Swarm is fully supported by Docker Engine, which means 1) it only takes a single line of command to create a Swarm and 2) it saves you time to mannually install docker-compose, which is not available in the standard Docker Engine.
* You are future-proofed. If you want to become highly-available and scale out your app, you won't need to start from scratch. Again, with Docker Swarm, it is just a few commands away from adding more nodes.
* docker-compose is only designed for development, not production, as it lacks a couple of important features out-of-the-box: 1) handling secret (that stores your keys and passwords securely) 2) auto-recovery of services, 3) rollbacks and 4) healtchecks. The last one is particularly crucial for production.
* You are able to do rolling update with Docker Swarm, which means no downtime for your app.
* Finally, if you are already using docker-compose.yml file, it is just a couple tweaks away to make it Docker Swarm friendly!

### Docker Swarm vs Kubernetes

In terms of container orchestration tools, Kubernetes is more popular. It covers almost all the use cases and can be more flexible than Docker Swarm. Plus, many vendors adopt the 'Kubernetes first' support strategy and some clouds even manage/deploy Kubernetes for you. However, I'd still argue that Docker Swarm is adequate for 80% of the use cases and much easier to set up. It appears that the 20-80 rule also apply here.

### Traefik vs Nginx

If you have read my previous post [Securing and monitoring ShinyProxy deployment of R Shiny apps]({{< ref "/post/secure-shinyproxy/index.md" >}}), you may wonder why I switched away from Nginx to Traefik. This is mainly due to the ease of set up. Nginx settings can end up in huge config maps that are hard to read and manage. This is not an issue with Traefik, which allows you to use Docker labels to manage the config. We will see this later in the tutorial.

## Prerequisites

In order to go through the tutorial, you need at least one server with the following specs:

* Ubuntu Server 18.04 LTS LTS (although other versions may also work)
* Minimal 1 GiB of memory
* Minimal 8 GB of storage
* Install Docker Engine

If you have an AWS Free Tier account, a free t2.micro instance would be fine. However, I'd recommend you go for something with 2 GiB of memory (e.g. t3a.small) if you also want to try out the monitoring / managing tools (e.g. Swarmpit, Grafana, etc.) mentioned later in this tutorial. My experience is that ShinyProxy uses about 200-300 MiB of memory and the Shiny app container uses about 100 MiB. On top of that, if you count the Traefik stack, you probably don't have enough remaining memory for Swarmpit.

You also need to set the relevant ports so the Swarm nodes can communicate with each other and allow traffic to your app. You should use the AWS Security Group (or equivalent from other Clouds) for easy set up and management. Below are the specific settings:

Swarm Manager Security Group (Inbound Rules):

| TYPE            | PROTOCOL | PORTS | SOURCE                     |
|-----------------|----------|-------|----------------------------|
| Custom TCP Rule | TCP      | 2377  | Swarm managers and workers |
| Custom TCP Rule | TCP      | 7946  | Swarm managers and workers |
| Custom UDP Rule | UDP      | 7946  | Swarm managers and workers |
| Custom UDP Rule | UDP      | 4789  | Swarm managers and workers |
| Custom Protocol | 50       | all   | Swarm managers and workers |
| SSH             | TCP      | 22    | Your ip                    |
| HTTP            | TCP      | 80    | Anywhere                   |
| HTTPS           | TCP      | 443   | Anywhere                   |

Swarm Worker Security Group (Inbound Rules):

| TYPE            | PROTOCOL | PORTS | SOURCE                     |
|-----------------|----------|-------|----------------------------|
| Custom TCP Rule | TCP      | 7946  | Swarm managers and workers |
| Custom UDP Rule | UDP      | 7946  | Swarm managers and workers |
| Custom UDP Rule | UDP      | 4789  | Swarm managers and workers |
| Custom Protocol | 50       | all   | Swarm managers and workers |
| SSH             | TCP      | 22    | Your ip                    |

We will first set up a manager node. Once you have launched the instance with the relevant ports opened, we will install Docker Engine using the set up script.

```{sh}
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

After installing Docker, I'd suggest that you add your user to the "docker" group so that you could use Docker as a non-root user.

```{sh}
sudo usermod -aG docker ubuntu
```

And don't forget the log out and back in for this change to take effect. Then you should be able to see the whole docker information without using `sudo`.

## Setting up Docker Swarm

As mentioned, you just need one line of command to initiate Docker Swarm, as it is built in the standard Docker Engine.

```{sh}
docker swarm init
```

You will see something like this:

```{sh}
Swarm initialized: current node (xxxx) is now a manager.
```

We then need to get the join token for managers and workers.

```{sh}
docker swarm join-token worker
docker swarm join-token manager
```

Note down the join commands. To add nodes to the current Swarm as a manager or worker, you simply need to launch another instance, install Docker Engine and run the join commands. However, we don't need to set them up for now.

## Setting up domains

Let's say you own the domain example.com and you want to use the subdomain app.example.com for your app. You need to create the following DNS records:

| RECORD TYPE | NAME                        | VALUE                            |
|-------------|-----------------------------|----------------------------------|
| A           | app.example.com             | IP of your Swarm Master instance |
| A           | traefik.sys.app.example.com | IP of your Swarm Master instance |
|             |                             | IP of your Swarm Master instance |
|             |                             | IP of your Swarm Master instance |
|             |                             | IP of your Swarm Master instance |
|             |                             |                                  |
|             |                             |                                  |

## Traefik Stack

Our next task is to set up the proxy/load balancer Traefik. [Docker Swarm Rocks](https://dockerswarm.rocks/traefik/) has a wonderful tutorial for it. I have summarised the key steps here. First, we need to create an `overlay` network shared with Traefik and allow nodes on the Swarm to communicate with each other. Note that this is different from the host-specific networks we create using the default `bridge` driver, which only allows networking between containers in one server. The `overlay` network sits on top of (overlays) the host-specific networks and allows containers connected to it to communicate securely when encryption is enabled.

```{sh}
docker network create --driver=overlay traefik-public
```

Get the Swarm node ID of this node and store it in an environment variable.

```{sh}
export NODE_ID=$(docker info -f '{{.Swarm.NodeID}}')
```

Create a tag in this node, so that Traefik is always deployed to the same node and uses the same volume.

```{sh}
docker node update --label-add traefik-public.traefik-public-certificates=true $NODE_ID
```

Create an environment variable with your email, to be used for the generation of Let's Encrypt certificates.

```{sh}
export EMAIL=admin@example.com
```

Create an environment variable with the domain you want to use for the Traefik UI (user interface). If you specified a different domain name before, you need to update the below code accordingly. You will access the Traefik dashboard at this domain.

```{sh}
export DOMAIN=traefik.sys.example.com
```

Create an environment variable with a username (you will use it for the HTTP Basic Auth for Traefik and Consul UIs).

```{sh}
export USERNAME=admin
```

Create an environment variable that store the hashed password.

```{sh}
export HASHED_PASSWORD=$(openssl passwd -apr1)
```

```{sh}
echo $HASHED_PASSWORD
```

It will look like:

```{sh}
$apr1$89eqM5Ro$CxaFELthUKV21DpI3UTQO
```

## ShinyProxy Stack

## 