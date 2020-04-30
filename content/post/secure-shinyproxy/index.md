---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Securing ShinyProxy Deployment of Shiny Apps"
subtitle: ""
summary: ""
authors: [databentobox]
tags: [R, R Shiny, ShinyProxy]
categories: [R Shiny]
date: 2020-05-02T22:10:55+01:00
lastmod: 2020-05-02T22:10:55+01:00
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

A while ago I have written a [tutorial]({{< ref "/post/deploy-r-app-with-shinyproxy/index.md" >}}) on deploying R Shiny apps using ShinyProxy. Following it, you should be able to deploy a functional Shiny app on AWS EC2 instance that can serve multiple users simultaneously. However, the app is not really production quality due to a couple of flaws. First, the authentication method of ShinyProxy was set to 'Simple', meaning that the usernames and passwords were stored as plain text in the `application.yml`. Second, the communication between users and the AWS instance is not secured as we didn't set up the HTTPS. Third, we were using the default AWS domain, which is hardly user friendly. Finally, we need a proper function to log usage statistics and system metrics in order to maintain the apps in long-term. In this tutorial, I will address these four issues by extending the current deployment framework utilising a couple of easy-to-set-up services.

In nutshell, We will use [Certbot(Let's Encrypt)](https://certbot.eff.org/) together with a valid domain name to set up the HTTPS site and use [Nginx](https://www.nginx.com/) as proxy to route HTTP traffic to HTTPS. For authentication, we will use [AWS Cognito](https://aws.amazon.com/cognito/), a service provided by AWS that  can use OpenID Connect protocol, for which ShinyProxy supports. For logging, we will use [InfluxDB](https://www.influxdata.com/products/influxdb-overview/), an open-source time series database, to store the usage statistics from ShinyProxy. We will use [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/) to collect other metrics about the host machine and Docker containers and feed those to InfluxDB. Finally, we will use [Grafana](https://grafana.com/), a well-established database analytics and monitoring platform to visualise the metrics. This may sound like a lot but I will try to break it down into different parts. Also, you don't need to implement all of these in one go except for the AWS Cognito, which depends on setting up the HTTPS first. However, if you are not familiar with ShinyProxy, you'd find this tutorial make more sense after checking out my previous post - [Deploying R Shiny apps using ShinyProxy on Windows 10]({{< ref "/post/deploy-r-app-with-shinyproxy/index.md" >}}).

## Prerequisites

Before we diving into the tutorial, there are a couple of things that need to be done before hand which I won't cover here:

1. Building a functional R Shiny app.
2. Setting up a (sub)domain name.

## Setting up HTTPS (SSL / TLS)

## (Optional) Setting up AWS Congnito for authentication

## Setting up InfluxDB, Telegraf and Grafana for usage statistics logging