---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Deploying Secure and Scalable Streamlit Apps on AWS with Docker Swarm, Traefik and Keycloak"
subtitle: ""
summary: "This post provides a guide to effectively and securely scale your Streamlit apps in production with Docker Swarm and Traefik, with the optional of adding authentication service provided by Keycloak"
authors: [Yihui Fan]
tags: [Python, Streamlit, Dash, Docker, Traefik, Keycloak]
categories: [Python]
date: 2020-10-08T14:30:54+01:00
lastmod: 2020-10-08T14:30:54+01:00
featured: true
draft: false

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

## Background

Streamlit is a popular open-source framework for creating machine learning and visualisation apps in Python. Although it is fun making your own Streamlit apps, deploying a production-grade app can be quite painful. If you are a data scientist who just wants to get the work done but doesn't necessarily want to go down the DevOps rabbit hole, this tutorial offers a relatively straightforward deployment solution leveraging Docker Swarm and Traefik, with an option of adding user authentication with Keycloak. The first part of the tutorial is meant to be a gentle introduction to Docker. We will deploy a dockerised Streamilit demo app locally. The second part is about deploying the app in the cloud. These are two relatively standalone parts. You can skip the first section if you are already familiar with Docker.

## Deploying apps locally with Docker

### Step 1: Installing Docker

Simply follow the instruction on [this page](https://docs.docker.com/docker-for-windows/install/) to download the latest Docker Windows Desktop from Docker (Mac users please see [here](https://docs.docker.com/docker-for-mac/install/); for Linux users, this [installation script](https://get.docker.com/) is very handy, which we will use when deploying the apps on AWS instances). Docker is a really powerful tool but I will only cover the basic commands and tools in this post. If you are interested, please check the tutorials online.

Please note the following system requirement:

* Windows 10 64-bit: Pro, Enterprise, or Education (Build 15063 or later).
* Hyper-V and Containers Windows features must be enabled.
* The following hardware prerequisites are required to successfully run Client Hyper-V on Windows 10:
  * 64-bit processor with [Second Level Address Translation (SLAT)](https://en.wikipedia.org/wiki/Second_Level_Address_Translation)
  * 4GB system RAM
  * BIOS-level hardware virtualization support must be enabled in the BIOS settings. For more information, see [Virtualization](https://docs.docker.com/docker-for-windows/troubleshoot/#virtualization-must-be-enabled).

If you are using Windows 7 or older version of macOS, you can try the [Docker Toolbox](https://docs.docker.com/toolbox/toolbox_install_windows/). It will create a small Linux VM (VirtualBox). This VM hosts Docker Engine for you on your Windows system. If you have a newer OS, then chances are you can use its native virtualization and does not require VirtualBox to run Docker.

### Step 2: Building and running the Streamlit GAN demo app

Since this post is about deployment, we won't be creating Streamlit apps from scratch but using the [GAN Face Generator demo app](https://github.com/streamlit/demo-face-gan/) instead. The app calls on TensorFlow to generate photorealistic faces, using Nvidia's Progressive Growing of GANs and Shaobo Guan's Transparent Latent-space GAN method for tuning the output face's characteristics.

First, let's clone the demo repo.

```{sh}
git clone https://github.com/streamlit/demo-face-gan.git
```

The `demo-face-gan` folder contains the data and the trained GAN models (`pg_gan` and `tl_gan`), the app script `app.py` and the `requirement.txt`. Normally, we would create a virtual environment and install the modules specified in the `requirement.txt`. But let's do it in the Docker way! Docker provides the ability to package and run an application in a loosely isolated environment called a container. A Docker container is nothing but an environment virtualized during run-time to allow users to isolate applications from the system underpinning it. To spin off containers, we need Docker images, which are non-changeable files containing libraries, source code, tools and other files needed to run applications. Let's start by creating a Docker image for the demo app.

We will need to create a file called `Dockerfile`. You can think of it as a set of instructions or blueprint to build the image. Copy the `Dockerfile` below to the `demo-face-gan` folder. Note that I have included comments in the file to explain each part. This is probably one of the simplest Dockerfile. For details of other options available in Dockerfiles, please see the [official document](https://docs.docker.com/engine/reference/builder/).

<details>
<summary>Show GAN App Dockerfile</summary>
<p>

```{Dockerfile}
# Dockerfile to create a Docker image for the demo-face-gan Streamlit app

# Creates a layer from the python:3.7 Docker image
FROM python:3.7

# Copy all the files from the folders the Dockerfile is to the container root folder
COPY . .

# Install the modules specified in the requirements.txt
RUN pip3 install -r requirements.txt

# The port on which a container listens for connections
EXPOSE 8501

# The command that run the app
CMD streamlit run app.py
```

</p>
</details>

---

Make sure Docker Desktop (daemon) is running and you have cd to the directory where the Dockerfile is and then run:

```{sh}
docker build -f Dockerfile -t streamlit-demo:latest .
```

It will take a while to set up the image. Once it is done, we can run the demo in Docker by:

```{sh}
docker run -p 8501:8501 streamlit-demo
```

Visit http://localhost:8501 to view the app.

![Streamlit Demo](streamlit_demo.png)

## Deploying the demo app on the cloud with Docker Swarm

We have successfully run the app in a Docker container locally. But in order to make it easily accessible to other users, we will deploy it to the cloud using a container orchestration framework called Docker Swarm. You can think of it as a cluster of machines on the cloud that hosts your app. You are probably wondering why you need Docker Swarm if your app is hosted on a single server. Docker Captain Bret Fisher explained it well [here](https://github.com/BretFisher/ama/issues/8). I summarised a couple of key benefits of Docker Swarm below:

* Docker Swarm is fully supported by Docker Engine, which means it only takes a single line of command to create a Swarm.
* You are future-proofed. If you want to scale out your app, you won't need to start from scratch. Again, with Docker Swarm, it is just a few commands away from adding more nodes.
* The vanilla Docker run and docker-compose is only designed for development, not production, as it lacks a couple of important features out-of-the-box: 1) handling secret (that stores your keys and passwords securely) 2) auto-recovery of services, 3) rollbacks and 4) healthchecks. The last one is particularly crucial for production.
* You are able to do rolling updates with Docker Swarm, which means no downtime for your app.
* Finally, if you are already using docker-compose.yml file, it is just a couple tweaks away to make it Docker Swarm friendly!

In terms of container orchestration tools, Kubernetes is more popular. It covers almost all the use cases and can be more flexible than Docker Swarm. Plus, many vendors adopt the 'Kubernetes first' support strategy and some clouds even manage/deploy Kubernetes for you. However, I'd still argue that Docker Swarm is adequate for 80% of the use cases and way much easier to set up. This means you can have your app running in hours rather than days!

We will put our app behind Traefik, which is a reverse proxy/load balancer. If you have read my previous post [Securing and monitoring ShinyProxy deployment of R Shiny apps]({{< ref "/post/secure-shinyproxy/index.md" >}}), you probably wonder why I switched away from Nginx to Traefik. This is mainly due to the ease of set up. Nginx settings can end up in huge config maps that are hard to read and manage. This is not an issue with Traefik, which allows you to use Docker labels to manage configs. We will see this later in the tutorial.

### Step 1: Setting up the manager node

 There are many cloud computing providers. In this tutorial, I will use AWS EC2 but the following steps can be easily implemented in other platforms. First, please refer to [this post](https://towardsdatascience.com/how-to-host-a-r-shiny-app-on-aws-cloud-in-7-simple-steps-5595e7885722) for launching an AWS EC2 instance if you are new to EC2. Since the app is actually quite computationally intensive, I chose the `t3a.medium` instance (2 vCPU, 4 GiB memory) and pick the `Ubuntu Server 18.04 LTS (HVM), SSD Volume Type` AMI. Unfortunately, this is not eligible for the free tier and will incur some cost (at $0.0376 per hour). You can switch to your own app, which may need less resources compared to the GAN demo app. But you may still want to deploy on an instance with at least 2GiB of memory due to other services that we need for this tutorial.

You also need to open the relevant ports so the Swarm nodes can communicate with each other and allow traffic to your app. You should use the AWS Security Group (or equivalent from other Clouds) for easy setup and management. Below are the specific settings:

<details>
<summary>SHOW AWS security group settings</summary>
<p>

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

</p>
</details>

---

We will first set up a manager node. Once you have launched the instance with the relevant ports opened, we will install Docker Engine using the setup script.

```{sh}
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

After installing Docker, I'd suggest that you add your user to the 'docker' group so that you could use Docker as a non-root user.

```{sh}
sudo usermod -aG docker ubuntu
```

And don't forget to log out and log back in again for this change to take effect. Then you should be able to run docker commands without using `sudo`.

### Step 2: Setting up Docker Swarm

As mentioned, you just need one line of command to initiate a Docker Swarm, as it is built into the standard Docker Engine.

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

### Step 3: Setting up domains for your app and system dashboards

Let's say you own the domain `example.com` and you want to use the subdomain `app.example.com` for your app. You need to create the following DNS records for your app and Traefik dashboard:

| RECORD TYPE | NAME                        | VALUE                            |
|-------------|-----------------------------|----------------------------------|
| A           | app.example.com             | IP of your Swarm Master instance |
| A           | traefik.sys.app.example.com | IP of your Swarm Master instance |

### Step 4: Setting up Traefik stack

First, let's set up the Traefik reserved proxy/load balancer. [Docker Swarm Rocks](https://dockerswarm.rocks/traefik/) has a wonderful tutorial for it. I have summarised the key steps here. First, we need to create an `overlay` network shared with Traefik and allow nodes on the Swarm to communicate with each other. Note that this is different from the host-specific networks we create using the default `bridge` driver, which only allows networking between containers in one server. The `overlay` network sits on top of (overlays) the host-specific networks and allows containers connected to it to communicate securely when encryption is enabled.

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

Create an environment variable with the domain you want to use for the Traefik dashboard. If you specified a different domain name before, you need to update the below code accordingly. You will access the Traefik dashboard at this domain.

```{sh}
export DOMAIN=traefik.sys.app.example.com
```

Create an environment variable with a username (you will use it for the HTTP Basic Auth for Traefik dashboard).

```{sh}
export USERNAME=admin
```

Create an environment variable that stores the hashed password. Note that the below command will allow you to enter the password into an interactive prompt, which is safer just typing into the shell (which will be stored in the shell history).

```{sh}
export HASHED_PASSWORD=$(openssl passwd -apr1)
```

Check if you have successfully created a password:

```{sh}
echo $HASHED_PASSWORD
```

It will look like:

```{sh}
$apr1$HOr/xJFw$uUY15r1qS.5AA2hk.ssda1
```

Now, let's deploy the first stack - Traefik. The author at Docker Swarm Rocks did an amazing job of making this process as easy as possible. You simply need to download the yaml file.

```{sh}
curl -L dockerswarm.rocks/traefik.yml -o traefik.yml
```

I have copied the `traefik.yml` file below with some comments inline that tell show what each part of code does. This is a fairly standard setup and we don't need to change anything for this tutorial.

<details>
<summary>SHOW traefik.yml</summary>
<p>

```{yml}
version: '3.3'

services:

  traefik:
    # Use the latest Traefik image
    image: traefik:v2.2
    ports:
      # Listen on port 80, default for HTTP, necessary to redirect to HTTPS
      - 80:80
      # Listen on port 443, default for HTTPS
      - 443:443
    deploy:
      placement:
        constraints:
          # Make the traefik service run only on the node with this label
          # as the node with it has the volume for the certificates
          - node.labels.traefik-public.traefik-public-certificates == true
      labels:
        # Enable Traefik for this service, to make it available in the public network
        - traefik.enable=true
        # Use the traefik-public network (declared below)
        - traefik.docker.network=traefik-public
        # Use the custom label "traefik.constraint-label=traefik-public"
        # This public Traefik will only use services with this label
        # That way you can add other internal Traefik instances per stack if needed
        - traefik.constraint-label=traefik-public
        # admin-auth middleware with HTTP Basic auth
        # Using the environment variables USERNAME and HASHED_PASSWORD
        - traefik.http.middlewares.admin-auth.basicauth.users=${USERNAME?Variable not set}:${HASHED_PASSWORD?Variable not set}
        # https-redirect middleware to redirect HTTP to HTTPS
        # It can be re-used by other stacks in other Docker Compose files
        - traefik.http.middlewares.https-redirect.redirectscheme.scheme=https
        - traefik.http.middlewares.https-redirect.redirectscheme.permanent=true
        # traefik-http set up only to use the middleware to redirect to https
        # Uses the environment variable DOMAIN
        - traefik.http.routers.traefik-public-http.rule=Host(`${DOMAIN?Variable not set}`)
        - traefik.http.routers.traefik-public-http.entrypoints=http
        - traefik.http.routers.traefik-public-http.middlewares=https-redirect
        # traefik-https the actual router using HTTPS
        # Uses the environment variable DOMAIN
        - traefik.http.routers.traefik-public-https.rule=Host(`${DOMAIN?Variable not set}`)
        - traefik.http.routers.traefik-public-https.entrypoints=https
        - traefik.http.routers.traefik-public-https.tls=true
        # Use the special Traefik service api@internal with the web UI/Dashboard
        - traefik.http.routers.traefik-public-https.service=api@internal
        # Use the "le" (Let's Encrypt) resolver created below
        - traefik.http.routers.traefik-public-https.tls.certresolver=le
        # Enable HTTP Basic auth, using the middleware created above
        - traefik.http.routers.traefik-public-https.middlewares=admin-auth
        # Define the port inside of the Docker service to use
        - traefik.http.services.traefik-public.loadbalancer.server.port=8080
    volumes:
      # Add Docker as a mounted volume, so that Traefik can read the labels of other services
      - /var/run/docker.sock:/var/run/docker.sock:ro
      # Mount the volume to store the certificates
      - traefik-public-certificates:/certificates
    command:
      # Enable Docker in Traefik, so that it reads labels from Docker services
      - --providers.docker
      # Add a constraint to only use services with the label "traefik.constraint-label=traefik-public"
      - --providers.docker.constraints=Label(`traefik.constraint-label`, `traefik-public`)
      # Do not expose all Docker services, only the ones explicitly exposed
      - --providers.docker.exposedbydefault=false
      # Enable Docker Swarm mode
      - --providers.docker.swarmmode
      # Create an entrypoint "http" listening on address 80
      - --entrypoints.http.address=:80
      # Create an entrypoint "https" listening on address 80
      - --entrypoints.https.address=:443
      # Create the certificate resolver "le" for Let's Encrypt, uses the environment variable EMAIL
      - --certificatesresolvers.le.acme.email=${EMAIL?Variable not set}
      # Store the Let's Encrypt certificates in the mounted volume
      - --certificatesresolvers.le.acme.storage=/certificates/acme.json
      # Use the TLS Challenge for Let's Encrypt
      - --certificatesresolvers.le.acme.tlschallenge=true
      # Enable the access log, with HTTP requests
      - --accesslog
      # Enable the Traefik log, for configurations and errors
      - --log
      # Enable the Dashboard and API
      - --api
    networks:
      # Use the public network created to be shared between Traefik and
      # any other service that needs to be publicly available with HTTPS
      - traefik-public

volumes:
  # Create a volume to store the certificates, there is a constraint to make sure
  # Traefik is always deployed to the same Docker node with the same volume containing
  # the HTTPS certificates
  traefik-public-certificates:

networks:
  # Use the previously created public network "traefik-public", shared with other
  # services that need to be publicly available via this Traefik
  traefik-public:
    external: true
```

</p>
</details>

---

When you have the file in your server, cd to the file directory and use the following command to deploy a Docker Swarm stack.

```{sh}
docker stack deploy -c traefik.yml traefik
```

There is only one service in this stack. You can check the status of this service using:

```{sh}
docker service ls
```

You will see something like below:

```{sh}
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
moybzwb7mq15        traefik_traefik     replicated          1/1                 traefik:v2.2        *:80->80/tcp, *:443->443/tcp
```

It is named as `traefik_traefik` because it is deployed into a stack called `traefik` and the service name is also called `traefik`. You can customise them if you like. Also, note that the `REPLICAS` variable shows you the number of copy of this service. '1/1' means we want only one copy and there is one up and running. You can check the log using:

```{sh}
docker service logs traefik_traefik
```

A few minutes after deploying the stack, Traefik should set up the SSL certificate for your site using Let' Encrypt. You may find this is much easier and cleaner than my previous solution. Now, check out `traefik.sys.app.example.com`. You should see the Traefik dashboard (use the username and password you just set to log in).

![Traefik Dashboard 1](traefik_dashboard_1.png)

Traefik is fairly complicated but we only need to use some of its functions so it is still manageable. There are some key concepts of Traefik, which I have summarised below:

* **Providers**: Discover the services that live on your infrastructure (their IP, health, ...). We are using Docker here.
* **Entrypoints**: Listen for incoming traffic (ports, ...). We have the 80 and 443 open for HTTP and HTTPS traffic.
* **Routers**: Analyse the requests (host, path, headers, SSL, ...). Currently, we only route relevant to `traefik.sys.app.example.com`. We can set up other routers later.
* **Services**: Forward the request to your services (load balancing, ...).
* **Middleware**: May update the request or make decisions based on the request (authentication, rate limiting, headers, ...)

All the things above can be created using the commands and labels in the `traefik.yml` file. For details, you may want to check the [official Traefik documentation](https://docs.traefik.io/).

### Step 5: Setting up the Streamlit GAN demo app stack

Now that we have set up the reverse proxy and load balancer, we can simply launch the demo Streamlit GAN app as a service in the Swarm. Please clone [my GitHub repo](https://github.com/presstofan/python-app-docker-swarm-stack).

```{sh}
git clone https://github.com/presstofan/python-app-docker-swarm-stack.git
cd python-app-docker-swarm-stack
```

The `stapp.yml` file is what we need. This yml file instructs Docker to set up a service in the Swarm. Unlike containers, a service can have many replicas (containers from the same image) to make sure the demand for the app can be met. These replicas will be deployed to different nodes of the Docker Swarm cluster and Traefik will make sure they are load-balanced. I have copied the `stapp.yml` file below and there is something you should know:

1. There is only one service in this stack, which is called `stapp`.
2. `image` is set to point to the Streamlit demo app uploaded to my Docker Hub repo. Using Docker Hub (or other similar services) to store images makes the deployment workflow much easier. If you want to deploy your own app, simply register a Docker Hub account and push the local image to it. You can find the quick start guide [here](https://docs.docker.com/docker-hub/).
3. The port for the `stapp` service is set to 8501. This is because when we create the image, we set it to open this port as default. It could be something else but we need to change the Dockerfile for the `streamlit-demo` image accordingly. Note that the port under the labels also need to be updated.
4. The overlay network `traefik-public` is used by the Traefik service, which will route the public HTTP/HTTPS requests at the host (app.example.com) to relevant services.
5. I commented out the `- node.role==manager` under the `placement: constraints:`. Because this service is the app itself, we don't need to limit it to the manager node. When we launch new workers, we want the load balancer to work and deploy the replicas of the app service to the new workers. I actually prefer to set it up to only deploy on worker nodes and let the manager node to handle just the Traefik stack. In this way, we can avoid the manager being overloaded with app requests. To do that, you need to set it to `- node.role==worker`.
6. `APP_DOMAIN` is what we set up earlier (e.g. app.example.com). If you want to host multiple apps you could set up different domains (e.g. app1.example.com, app2.example.com). You then need to set up a router for each app and specify the domain in the labels section of that service (e.g. replacing `APP_DOMAIN`). Traefik will route the visitors to different app services based on the domain specified.
7. Note that `replicas` has been set up to one. You may want to tweak it to find the right number given your Swarm cluster and the demand. Of course, you can scale it up and down afterwards.

<details>
<summary>SHOW stapp.yml</summary>
<p>

```{yml}
version: '3.3'

services:

  stapp:
    image: presstofan/streamlit-demo
    # The labels section is where you specify configuration values for Traefik.
    # Docker labels don’t do anything by themselves, but Traefik reads these so
    # it knows how to treat containers.
    ports:
      - 8501
    networks:
      - traefik-public
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure
      # placement:
      #   constraints:
      #     - node.role==manager
      labels:
          - traefik.enable=true # enable traefik
          - traefik.docker.network=traefik-public # put it in the same network as traefik
          - traefik.constraint-label=traefik-public # assign the same label as traefik so it can be discovered
          - traefik.http.routers.stapp.rule=Host(`${APP_DOMAIN?Variable not set}`) # listen to port 80 for request to APP_DOMAIN (use together with the line below)
          - traefik.http.routers.stapp.entrypoints=http
          - traefik.http.middlewares.stapp.redirectscheme.scheme=https # redirect traffic to https
          - traefik.http.middlewares.stapp.redirectscheme.permanent=true # redirect traffic to https
          - traefik.http.routers.stapp-secured.rule=Host(`${APP_DOMAIN?Variable not set}`) # listen to port 443 for request to APP_DOMAIN (use together with the line below)
          - traefik.http.routers.stapp-secured.entrypoints=https
          - traefik.http.routers.stapp-secured.tls.certresolver=le # use the Let's Encrypt certificate we set up earlier
          - traefik.http.services.stapp-secured.loadbalancer.server.port=8501 # ask Traefik to search for port 8501 of the stapp service container
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock

networks:
  traefik-public:
    external: true
```

</p>
</details>

---

Before deploying the app service, let's set up the environment variable `APP_DOMAIN`. This should be the domain of your app you set up earlier with your DNS provider (e.g. app.example.com). When users visiting this domain, Traefik will route it to the app service.

```{sh}
export APP_DOMAIN=app.example.com
```

Then, let's deploy the service with the `stapp.yml` file and call it `app`.

```{sh}
docker stack deploy -c stapp.yml app
```

We can check the status of the service using:

```{sh}
docker service ls
```

You will see the following services (it can take a few minutes to download the images for the first time):

```{sh}
ID                  NAME                MODE                REPLICAS            IMAGE                              PORTS
leqx49lfktgl        app_stapp           replicated          1/1                 presstofan/streamlit-demo:latest   *:30000->8501/tcp
2eu098a9usi6        traefik_traefik     replicated          1/1                 traefik:v2.2                       *:80->80/tcp, *:443->443/tcp
```

Give it a minute and check `app.example.com` and you will see the `streamlit-demo` app. If you go to the Traefik dashboard, you can now find the additional router, service and middleware related to the Streamlit app.

This is only one replica of this service so users will share the resource from the same container. We can scale up the number of replicas:

```{sh}
docker service update app_stapp --replicas 2
```

Now if you run:

```{sh}
docker stats
```

You will see two containers:

```{sh}
CONTAINER ID        NAME                                          CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
847576f23b02        app_stapp.2.tcwdyysex8sh346zncbor5m3l         2.09%               580.9MiB / 3.797GiB   14.94%              187MB / 819kB       11.5MB / 0B         21
9017e6efae0d        app_stapp.1.qs5nv8w6pl2vy26ugmmf9g18p         2.36%               1.133GiB / 3.797GiB   29.82%              187MB / 7.13MB      12MB / 0B           21
552b874cbe5d        traefik_traefik.1.34r1hky0knrxw6eglwosby1zw   0.04%               15.27MiB / 3.797GiB   0.39%               7.02MB / 9.5MB      213kB / 45.1kB      8
```

And now if you visit `app.example.com`, you will be routed to one of the two copies of the app.

There are many things we can tweak using the `service update`, such as CPU and memory limit. Please see [here](https://docs.docker.com/engine/reference/commandline/service_update/) for details.

### Step 6: Scaling your Swarm cluster

Now comes to the interesting part. Let's say if your app is getting popular and you want to launch an additional server to share the workload. You can easily add nodes to your Swarm.

First, we need to launch another AWS instance (or server from other Clouds). Repeat the steps above but this time we want to use the security group settings for Swarm Workers.

<details>
<summary>SHOW AWS security group settings</summary>
<p>

Swarm Worker Security Group (Inbound Rules):

| TYPE            | PROTOCOL | PORTS | SOURCE                     |
|-----------------|----------|-------|----------------------------|
| Custom TCP Rule | TCP      | 7946  | Swarm managers and workers |
| Custom UDP Rule | UDP      | 7946  | Swarm managers and workers |
| Custom UDP Rule | UDP      | 4789  | Swarm managers and workers |
| Custom Protocol | 50       | all   | Swarm managers and workers |
| SSH             | TCP      | 22    | Your ip                    |

</p>
</details>

---

SSH into the new instance and install Docker Engine as above. When that is done, we need to join the Swarm we set up and the join token we got from the Manager node would come in handy. Run it in the new instance. The token would look like something below:

```{sh}
docker swarm join --token SWMTKN-1-xxxxxxxxxxxxxxxxxxx-xxxxxxxxxxxxxxx 172.x.x.x:2377
```

If successful, you would see the message 'This node joined a swarm as a worker.' If you now switch to the Manager node shell and run:

```{sh}
docker node ls
```

You would see:

```{sh}
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
5b6py13brdoihrcct9oy68wpq *   ip-172-31-49-53     Ready               Active              Leader              19.03.10
3j3ecsf4fhz3cwhu98w5cv1bo     ip-172-31-71-189    Ready               Active                                  19.03.10
```

Congratulations! You have just created your two-node Swarm. Traefik only gets deployed on the Manager node as we instructed but the the Streamlit app can be deployed to worker nodes (or only on the worker node if you specified so in the yml file). Swarm will manage the load automatically. You can use `docker service ps SERVICE_NAME` to check the node the service is on. Note that the first time you deploy a service to a node it will take some time to download the images. But it won't be a problem later on as it should have the images cached. For example, you may see the two containers of the app are running on different nodes as shown below:

```{sh}
ID                  NAME                IMAGE                              NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
m4rb4xf1lzth        app_stapp.1         presstofan/streamlit-demo:latest   ip-172-31-3-100     Running             Running 13 minutes ago
bf6ibptukrsz        app_stapp.2         presstofan/streamlit-demo:latest   ip-193-21-6-300     Running             Running 8 minutes ago
```

#### Choosing the number of nodes

Here are some notes on choosing the number of nodes. In theory, you can set up more than one Manager nodes (e.g. three or even five). However, the community edition of Traefik won't support distributed Let's Encrypt certificate, meaning that only one of the node can be your gateway and your DNS need to point the domain to that node. If you are building a complex app that requires HA, you may want to consider [Traefik Enterprise Edition](https://containo.us/traefikee/). For many use cases, making your primary Manager node sufficiently powerful (e.g. at least 2 GiB of memory, preferable 4 GiB) and offloading Shiny apps to the workers would be good enough. There is less of a constraint in choosing the number of worker nodes (although preferably an even number) and the specs depend on the app you serve.

#### Rebalancing nodes

When you add a new node to a Swarm or a node reconnects to the Swarm after a period of unavailability, the Swarm does not automatically give a workload to the idle nodes. According to Docker, this is a design decision.

> If the swarm periodically shifted tasks to different nodes for the sake of balance, the clients using those tasks would be disrupted. The goal is to avoid disrupting running services for the sake of balance across the swarm. When new tasks start, or when a node with running tasks becomes unavailable, those tasks are given to less busy nodes. The goal is eventual balance, with minimal disruption to the end user.

If needed, we could force the nodes to rebalance by using the command below:

```{sh}
docker service update --force app_stapp
```

This is pretty handy if you have just added or removed many nodes. However, the update causes service tasks to restart. Client applications may be disrupted.

### (Optional) Step 7: Adding a Keycloak authentication layer

Sometimes we may want to protect the app behind a user authentication layer. This can be achieved via many authentication providers. In this tutorial, I will be using [Keycloak](https://www.keycloak.org/about), an open-source Identity and Access Management solution. I have previously shown how you can [add authentication to ShinyProxy using AWS Cognito]({{< ref "/post/shinyproxy-with-docker-swarm/index.md" >}}). Using Keycloak will give us more control. In a nutshell, we will create a Keycloak server which is used for managing users and providing authentication. We will then add an authentication client in front of our Streamlit app. Ideally, the Keycloak server and the app should be hosted by different AWS instances for security and reliability but for the sake of this tutorial, we will deploy both of them on the same instance.

#### Setting up Keycloak server

First, we need to create another DNS record for the Keycloak console.

| RECORD TYPE | NAME                         | VALUE                            |
|-------------|------------------------------|----------------------------------|
| A           | keycloak.sys.app.example.com | IP of your Swarm Master instance |

Then store the domain name for later use:

```{sh}
export KEYCLOAK_DOMAIN=keycloak.sys.app.example.com
```

The following Keycloak setup is largely based on this wonderful [recipe](https://geek-cookbook.funkypenguin.co.nz/recipes/keycloak/). I have modified it to work with the existing Traefik stack. First, let's create two folders which we will later use to store the Keycloak database and backups.

```{sh}
sudo mkdir -p /var/data/runtime/keycloak/database
sudo mkdir -p /var/data/keycloak/database-dump
```

Clone the tutorial repo. Note that you may have already done this step if you have followed the first part of this tutorial. If that is the case, simply cd into the directory.

```{sh}
git clone https://github.com/presstofan/python-app-docker-swarm-stack.git
cd python-app-docker-swarm-stack
```

In this directory, you will find the yml file for the Keycloak stack.

<details>
<summary>SHOW keycloak.yml</summary>
<p>

```{yml}
version: '3'

services:
  keycloak:
    image: jboss/keycloak
    depends_on:
      - keycloak-db
    environment:
      - DB_VENDOR=postgres
      - DB_DATABASE=keycloak
      - DB_ADDR=keycloak-db
      - DB_USER=keycloak
      - DB_PASSWORD=changeme
      - KEYCLOAK_USER=admin
      - KEYCLOAK_PASSWORD=changeme
      - PROXY_ADDRESS_FORWARDING=true # This is required to run keycloak behind traefik
      - KEYCLOAK_HOSTNAME=${KEYCLOAK_DOMAIN?Variable not set}
      - POSTGRES_USER=keycloak
      - POSTGRES_PASSWORD=changeme
    volumes:
      - /etc/localtime:/etc/localtime:ro
    networks:
      - traefik-public
      - keycloak-internal
    deploy:
      labels:
        - traefik.enable=true # enable traefik
        - traefik.docker.network=traefik-public # put it in the same network as traefik
        - traefik.constraint-label=traefik-public # assign the same label as traefik so it can be discovered
        - traefik.http.routers.keycloak.rule=Host(`${KEYCLOAK_DOMAIN?Variable not set}`) # listen to port 80 for request to KEYCLOAK_DOMAIN (use together with the line below)
        - traefik.http.routers.keycloak.entrypoints=http
        - traefik.http.middlewares.keycloak-https-redirect.redirectscheme.scheme=https # redirect traffic to https
        - traefik.http.middlewares.keycloak-https-redirect.redirectscheme.permanent=true # redirect traffic to https
        - traefik.http.routers.keycloak-secured.rule=Host(`${KEYCLOAK_DOMAIN?Variable not set}`) # listen to port 443 for request to KEYCLOAK_DOMAIN (use together with the line below)
        - traefik.http.routers.keycloak-secured.entrypoints=https
        - traefik.http.routers.keycloak-secured.tls.certresolver=le # use the Let's Encrypt certificate we set up earlier
        - traefik.http.services.keycloak-secured.loadbalancer.server.port=8080 # ask Traefik to search for port 8080 of the Keycloak service container

  keycloak-db:
    image: postgres:10.1
    environment:
      - DB_VENDOR=postgres
      - DB_DATABASE=keycloak
      - DB_ADDR=keycloak-db
      - DB_USER=keycloak
      - DB_PASSWORD=changeme
      - KEYCLOAK_USER=admin
      - KEYCLOAK_PASSWORD=changeme
      - PROXY_ADDRESS_FORWARDING=true # This is required to run keycloak behind traefik
      - KEYCLOAK_HOSTNAME=${KEYCLOAK_DOMAIN?Variable not set}
      - POSTGRES_USER=keycloak
      - POSTGRES_PASSWORD=changeme
      - KEYCLOAK_LOGLEVEL=WARNING
    volumes:
      - /var/data/runtime/keycloak/database:/var/lib/postgresql/data
      - /etc/localtime:/etc/localtime:ro
    networks:
      - keycloak-internal

  keycloak-db-backup:
    image: postgres:10.1
    environment:
      - PGHOST=keycloak-db
      - PGUSER=keycloak
      - PGPASSWORD=changeme
      - BACKUP_NUM_KEEP=7
      - BACKUP_FREQUENCY=1d
    volumes:
      - /var/data/keycloak/database-dump:/dump
      - /etc/localtime:/etc/localtime:ro
    entrypoint: |
      bash -c 'bash -s <<EOF
      trap "break;exit" SIGHUP SIGINT SIGTERM
      sleep 2m
      while /bin/true; do
        pg_dump -Fc > /dump/dump_\`date +%d-%m-%Y"_"%H_%M_%S\`.psql
        (ls -t /dump/dump*.psql|head -n $$BACKUP_NUM_KEEP;ls /dump/dump*.psql)|sort|uniq -u|xargs rm -- {}
        sleep $$BACKUP_FREQUENCY
      done
      EOF'
    networks:
      - keycloak-internal

networks:
  traefik-public:
    external: true
  keycloak-internal:
    driver: overlay
    ipam:
      config:
        # Setup unique static subnets for every stack you deploy. 
        # This avoids IP/gateway conflicts which can otherwise occur when you're creating/removing stacks a lot.
        - subnet: 172.16.49.0/24
```

</p>
</details>

---

Let's deploy the Keycloak server service with the `keycloak.yml` file and call it `keycloak`.

```{sh}
docker stack deploy -c keycloak.yml keycloak
```

We can check the status of the service using:

```{sh}
docker service ls
```

You will see two additional Keycloak services (again, it can take a few minutes to download the images for the first time):

```{sh}
ID                  NAME                          MODE                REPLICAS            IMAGE                   PORTS
xn43fgqs596d        keycloak_keycloak             replicated          1/1                 jboss/keycloak:latest
w3j4rmuwgk2p        keycloak_keycloak-db          replicated          1/1                 postgres:10.1
zmqu2zon25p1        keycloak_keycloak-db-backup   replicated          1/1                 postgres:10.1
2eu098a9usi6        traefik_traefik               replicated          1/1                 traefik:v2.2            *:80->80/tcp, *:443->443/tcp
```

Give it a minute and check `keycloak.sys.app.example.com` and you will see the Keycloak Welcome page. Log into the Administrative Console with the username and password specified in the docker-compose file. The default username and password are "admin" and "changeme" (if you haven't changed the yml file).

![Keycloak Login](keycloak_login.png)

#### Setting up Keycloak client and user

First, let's create a new client for the Streamlit app in Keycloak. In the `Master` realm, click `Clients` and then `Create`.

![Keycloak Create Client 1](keycloak_create_client_1.png)

We can call it `stapp` and use the `openid-connect` protocol.

![Keycloak Create Client 2](keycloak_create_client_2.png)

After hitting save, we need to change the Access Type to `confidential` and add `https://app.example.com/*` to Valid Redirect URIs. This is where you will be directed to after successfully logged in. Don't forget to change it your own domain. Save it again and go to the Credential tab and note down the `Secret`.

![Keycloak Create Client 3](keycloak_create_client_3.png)

Now, we need to set up a user for testing. click `Users` and then `Add User`.

![Keycloak Create User 1](keycloak_create_user_1.png)

Let's give it a name called `test`. After saving it, go to the Credential tab and set the password to "changeme" and save again.

![Keycloak Create User 1](keycloak_create_user_2.png)

#### Setting up Traefik Forward Auth

We will set up a Traefik Forward Auth service to take care of the authentication process. The user flow is like this: A user who visit `app.example.com` for the first time get redirected to the Traefik Forward Auth service, which will send the user to Keycloak for authentication. Once the user has logged in, she/he will be redirected back to Traefik Forward Auth service and it will check the authentication code. If successful, the user will be redirected to the app service.

First, we need to take down the existing `app` stack (which doesn't require authentication).

```{sh}
docker stack rm app
```

In the `python-app-docker-swarm-stack` folder, there is another file called `stapp-auth.yml`, which will help us to set up the new app stack bundling the Streamlit app and the Traefik Forward Auth service.

<details>
<summary>SHOW stapp-auth.yml</summary>
<p>

```{yml}
version: '3.3'

services:
  traefik-forward-auth:
    image: thomseddon/traefik-forward-auth:2
    environment:
      - DEFAULT_PROVIDER=oidc
      - PROVIDERS_OIDC_ISSUER_URL=https://${KEYCLOAK_DOMAIN?Variable not set}/auth/realms/master
      - PROVIDERS_OIDC_CLIENT_ID=${AUTH_CLIENT_ID?Variable not set}
      - PROVIDERS_OIDC_CLIENT_SECRET=${AUTH_CLIENT_SECRET?Variable not set}
      - SECRET=change_to_a_random_string
      # INSECURE_COOKIE is required if not using a https entrypoint
      # - INSECURE_COOKIE=true
      - LOG_LEVEL=debug
    networks:
      - traefik-public
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints:
           - node.role==manager
      labels:
        - "traefik.enable=true" # enable traefik
        - "traefik.docker.network=traefik-public" # put it in the same network as traefik
        - "traefik.constraint-label=traefik-public" # assign the same label as traefik so it can be discovered
        - "traefik.http.middlewares.traefik-forward-auth.forwardauth.address=http://traefik-forward-auth:4181"
        - "traefik.http.middlewares.traefik-forward-auth.forwardauth.authResponseHeaders=X-Forwarded-User"
        - "traefik.http.services.traefik-forward-auth.loadbalancer.server.port=4181"
        # receive the authentication results from Keycloak
        - "traefik.http.routers.forward-auth.rule=Host(`auth.${APP_DOMAIN?Variable not set}`)"
        - "traefik.http.routers.forward-auth.service=traefik-forward-auth"

  stapp:
    image: presstofan/streamlit-demo
    depends_on:
      - traefik-forward-auth
    ports:
      - 8501
    networks:
      - traefik-public
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure
      # Use below if only place the app on workers
      # placement:
      #   constraints:
      #     - node.role==worker
      labels:
        - "traefik.enable=true" # enable traefik
        - "traefik.docker.network=traefik-public" # put it in the same network as traefik
        - "traefik.constraint-label=traefik-public" # assign the same label as traefik so it can be discovered
        - "traefik.http.routers.stapp.rule=Host(`${APP_DOMAIN?Variable not set}`)"
        - "traefik.http.routers.stapp.entrypoints=http"
        # redirect HTTP to HTTPS and add SSL certificate
        - "traefik.http.middlewares.stapp-https-redirect.redirectscheme.scheme=https" # redirect traffic to https
        - "traefik.http.middlewares.stapp-https-redirect.redirectscheme.permanent=true" # redirect traffic to https
        - "traefik.http.routers.stapp-secured.rule=Host(`${APP_DOMAIN?Variable not set}`)" # listen to port 443 for request to APP_DOMAIN (use together with the line below)
        - "traefik.http.routers.stapp-secured.entrypoints=https"
        - "traefik.http.routers.stapp-secured.tls.certresolver=le" # use the Let's Encrypt certificate we set up earlier
        # assign app service
        - "traefik.http.routers.stapp-secured.service=stapp"
        - "traefik.http.routers.stapp-secured.middlewares=stapp-auth@docker"
        # redirect to Keycloak for authentication
        - "traefik.http.middlewares.stapp-auth.forwardauth.address=http://traefik-forward-auth:4181"
        - "traefik.http.middlewares.stapp-auth.forwardauth.authresponseheaders=X-Forwarded-User"
        - "traefik.http.services.stapp.loadbalancer.server.port=8501"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock

networks:
  traefik-public:
    external: true
```

</p>
</details>

---

Before deploying it, we need to set up a couple of environment variables (the client id and secret) which the yml refers to. Please change the secret below to your own.

```{sh}
export AUTH_CLIENT_ID=stapp
export AUTH_CLIENT_SECRET=77b5cf3b-4054-407f-b9d6-ba2c18cc5b70
```

Now, let's run:

```{sh}
docker stack deploy -c stapp-auth.yml app
```

Give it a minute and then visit `app.example.com`. You will see the Keycloak login page. Once logged in, you will be directed to the Streamlit app. Note that you can also log in with the admin account we created for Keycloak, but you can't use the test account to log in the Keycloak Administrative Console. This is determined by the role mapping of each user. There are many things you can do with Keycloak. For example, you can allow user self registration, which is a useful feature for some apps. Check the [official document](https://www.keycloak.org/docs/latest/server_admin/index.html) if you are interested.

### (Optional) Step 8: Monitoring Docker Swarm with Swarmpit

[Swarmpit](https://github.com/swarmpit/swarmpit) provides simple and easy to use interface for your Docker Swarm cluster. You can manage your stacks, services, secrets, volumes, networks etc. To set this up, we first need to create another "A" record with our DNS provider to point to the Manager node IP address.

| RECORD TYPE | NAME                        | VALUE                            |
|-------------|-----------------------------|----------------------------------|
| A           |swarmpit.sys.app.example.com | IP of your Swarm Master instance |

The process of deploying the Swarmpit stack is very similar to how we deployed the Traefik stack.

Set up the environment variables:

```{sh}
export DOMAIN=swarmpit.sys.app.example.com
export NODE_ID=$(docker info -f '{{.Swarm.NodeID}}')
```

Create a label in this node, so that the CouchDB database used by Swarmpit is always deployed to the same node and uses the existing volume:

```{sh}
docker node update --label-add swarmpit.db-data=true $NODE_ID
```

Create another label in this node, so that the Influx database used by Swarmpit is always deployed to the same node and uses the existing volume:

```{sh}
docker node update --label-add swarmpit.influx-data=true $NODE_ID
```

Download the `swarmpit.yml`

```{sh}
curl -L dockerswarm.rocks/swarmpit.yml -o swarmpit.yml
```

Or create one in the Manager node yourself with the template below:

<details>
<summary>SHOW swarmpit.yml</summary>
<p>

```{yml}
version: '3.3'

services:
  app:
    image: swarmpit/swarmpit:latest
    environment:
      - SWARMPIT_DB=http://db:5984
      - SWARMPIT_INFLUXDB=http://influxdb:8086
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    ports:
      - 888:8080
    networks:
      - net
      - traefik-public
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 1024M
        reservations:
          cpus: '0.25'
          memory: 512M
      placement:
        constraints:
          - node.role == manager
      labels:
        - traefik.enable=true
        - traefik.docker.network=traefik-public
        - traefik.constraint-label=traefik-public
        - traefik.http.routers.swarmpit-http.rule=Host(`${DOMAIN?Variable not set}`)
        - traefik.http.routers.swarmpit-http.entrypoints=http
        - traefik.http.routers.swarmpit-http.middlewares=https-redirect
        - traefik.http.routers.swarmpit-https.rule=Host(`${DOMAIN?Variable not set}`)
        - traefik.http.routers.swarmpit-https.entrypoints=https
        - traefik.http.routers.swarmpit-https.tls=true
        - traefik.http.routers.swarmpit-https.tls.certresolver=le
        - traefik.http.services.swarmpit.loadbalancer.server.port=8080

  db:
    image: couchdb:2.3.0
    volumes:
      - db-data:/opt/couchdb/data
    networks:
      - net
    deploy:
      resources:
        limits:
          cpus: '0.30'
          memory: 512M
        reservations:
          cpus: '0.15'
          memory: 256M
      placement:
        constraints:
          - node.labels.swarmpit.db-data == true
  influxdb:
    image: influxdb:1.7
    volumes:
      - influx-data:/var/lib/influxdb
    networks:
      - net
    deploy:
      resources:
        reservations:
          cpus: '0.3'
          memory: 128M
        limits:
          cpus: '0.6'
          memory: 512M
      placement:
        constraints:
          - node.labels.swarmpit.influx-data == true
  agent:
    image: swarmpit/agent:latest
    environment:
      - DOCKER_API_VERSION=1.35
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      - net
    deploy:
      mode: global
      resources:
        limits:
          cpus: '0.10'
          memory: 64M
        reservations:
          cpus: '0.05'
          memory: 32M

networks:
  net:
    driver: overlay
    attachable: true
  traefik-public:
    external: true

volumes:
  db-data:
    driver: local
  influx-data:
    driver: local
```

</p>
</details>

---

When ready, deploy the stack using:

```{sh}
docker stack deploy -c swarmpit.yml swarmpit
```

Check if it is running:

```{sh}
docker stack ps swarmpit
```

It will show something like below:

```{sh}
ID             NAME                       IMAGE                      NODE                DESIRED STATE   CURRENT STATE          ERROR   PORT
kkhasdfvce30   swarmpit_agent.ndasdfav5   swarmpit/agent:latest      dog.example.com     Running         Running 3 minutes ago
k8oasdfg70jm   swarmpit_agent.i9asdfjps   swarmpit/agent:latest      cat.example.com     Running         Running 3 minutes ago
kcvasdft0yzj   swarmpit_agent.3jasdfd3k   swarmpit/agent:latest      snake.example.com   Running         Running 3 minutes ago
9onasdfzopve   swarmpit_agent.r6asdfb20   swarmpit/agent:latest      snake.example.com   Running         Running 3 minutes ago
fxoasdfwjrbj   swarmpit_db.1              couchdb:2.3.0              dog.example.com     Running         Running 3 minutes ago
m4jasdf3369c   swarmpit_app.1             swarmpit/swarmpit:latest   cat.example.com     Running         Running 3 minutes ago
```

Finally, give it a minute or two and go to `swarmpit.sys.app.example.com` to access the dashboard. You will be prompted to set up an account the first time you use it.

![Swarmpit](swarmpit.png)

## Next Steps

That concludes this tutorial. We should have a scalable Streamlit app served by Docker Swarm with a secured host, authentication service and monitoring capability. Using Docker Swarm and Traefik makes our lives much easier and the deployment should be future-proofed. One thing that is nice to have is the ability to elastically scale the app, meaning that new AWS EC2 instances will be automatically deployed and join the Swarm cluster in response to the app demand. This proves to be a non-trivial task and I will continue to explore it in the future. But please post a comment below if you happen to know a good way of achieving it!
