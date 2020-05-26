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

If you search for R Shiny apps deployment and land on this post, chances are you are like me - a data scientist who just want to build a decent app to host a dashboard or a prediction model without going down to the rabbit hole of DevOps or the frontend / backend development. Don't worry, this is just the thing you need and I promise nothing below is too complicated. If you really focus, you can go through the following tutorial in half an hour. Let's start.

In this post, we will build a scalable, production-grade Shiny app powered by ShinyProxy and Docker Swarm and use Traefik to handle the SSL certificate (which gives you the little padlock before your domain name) and reverse proxy (i.e. route traffic from the 80 and 443 ports to your Shiny app and other places if needed). If you are not familiar with ShinyProxy and wonder why you should use it over the other options, I have written another post titled [Deploying R Shiny apps using ShinyProxy on Windows 10]({{< ref "/post/deploy-r-app-with-shinyproxy/index.md" >}}) that explains it in more detail. If you haven't used Docker technology before, it is also a good idea to check out that post first to get your hands on it. In the interest of space, I won't cover too much details regarding Docker in this tutorial.

### Docker Swarm vs standard Docker containers

You probably wonder why you need Docker Swarm rather than just using docker-compose to deploy containers, particularly if you only have one server/node/instance. Docker Captain Bret Fisher explained it well [here](https://github.com/BretFisher/ama/issues/8). I summarised a couple of key benefits of Docker Swarm below:

* Docker Swarm is fully supported by Docker Engine which means 1) it only takes a single line of command to create a Swarm and 2) it saves you time to mannually install docker-compose, which is not available in the standard Docker Engine.
* You are future-proofed. If you want to become highly-available and scale out your app, you won't need to start from scratch. Again, with Docker Swarm, it is just a few commands away from adding more nodes.
* docker-compose is only designed for development, not production, as it lacks a couple of important features out-of-the-box: 1) handling secret (that stores your keys and passwords securely) 2) auto-recovery of services, 3) rollbacks and 4) healtchecks. The last one is particularly crucial for production.
* You are able to do rolling update with Docker Swarm, which means no downtime for your app.
* Finally, if you are already using docker-compose.yml file, it is just a couple tweaks away to make it Docker Swarm friendly!

### Docker Swarm vs Kubernetes

In terms of container orchestration, Kubernetes is more popular. However, I'd argue that Docker Swarm is adequate for 80% of the use cases and much easier to set up. That said, Kubernetes covers more use cases and can be more flexible than Docker Swarm. And many vendors adopt the 'Kubernetes first' support strategy and some clouds even manage/deploy Kubernetes for you. This is for another post.

## Traefik Stack

## ShinyProxy Stack

## 