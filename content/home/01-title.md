+++
title = "Docker From Scratch"
outputs = ["Reveal"]
+++

{{< slide timing="180" >}}

# Docker From Scratch

Hands-on Workshop

{{% note %}}
<pre>
* Who am I?
* How many of you have used
  Docker?
* In production?

* Workshop Ground Rules:
  * Volunteers
  * Ask Questions
</pre>
{{% /note %}}

---

{{< slide timing="120" >}}

## What is Docker?

* A cloud infrastructure company (formerly dotCloud) that created Docker as an internal tool
* Container Runtime daemon that runs and manages Containers on top of Linux Kernel
* CLI client to the Docker Daemon
* Container Image Packaging Format, now evolved into OCI Image

{{% note %}}
5m

{{% /note %}}

---

{{< slide timing="300" >}}

## Containers vs Virtual Machines

![Containers and VMs](https://i0.wp.com/blog.docker.com/wp-content/uploads/Blog.-Are-containers-..VM-Image-1.png)

(Source: [Docker Blog: Are Containers Replacing Virtual Machines? by Jenny Fong](https://blog.docker.com/2018/08/containers-replacing-virtual-machines/))

{{% note %}}
10m

{{% /note %}}

---

{{< slide timing="300" >}}

## History of Containers

* **1979**: Development of `chroot` in Unix V7
* **2003**: Google Created its internal cluster management system Borg
* **2004**: Development Process Containers, which eventually became `cgroups`
* **2008**: `cgroups` got absorbed in Linux Kernel in 2008, which led to development of LXC
* **2013**: Docker was built on top of LXC
* **2015**: Open Container Initiative (OCI) was created by Docker, CoreOS and others to standardize containers. Docker donated its container runtime `runc` to the OCI. Kubernetes was born.
* **2017 Onwards**: Kubernetes emerged as a victor in Container Orchestrator Wars™. Seeds of alternative ecosystem were sewn with development of alternative tools like CRI-O, and Podman. Evolution is still ongoing.

{{% note %}}
<pre>
* Honorary mentions: BSD Jails, SmartOS
</pre>
{{% /note %}}

---

{{< slide timing="300" >}}

## Enough Talk. Let's Get Our Hands Dirty!

* Visit <app-link>, and authenticate yourself through the gmail account you used to register for the workshop

{{% note %}}
20m

{{% /note %}}

---

{{< slide timing="300" >}}

## Run a task

```bash
docker run busybox echo "Hello Laravel Rajkot"

# List running containers
docker ps

# List all containers
docker ps -a

# Remove Exited Container
docker rm <container-name>
```

{{% note %}}
<pre>
* docker run
* docker pull
* docker ps
* docker rename
* docker rm
</pre>
{{% /note %}}

---
## Run an interactive container

```bash
docker run -it --rm busybox sh

# List running processes
ps aux

# Exit from the container
exit

ps aux
```

{{% note %}}
6m

{{% /note %}}

---
## Run a background nginx container

```bash
# Visit the IP address of your docker host to see whether nginx is running

docker run -d -p 80:80 nginx:alpine

# Visit the IP address of your docker host to see nginx running

docker ps
# Check access logs
docker logs -f <nginx-container-name>

# Open Interactive Shell within a running container
docker exec -it <nginx-container-name> sh

# Navigate to the location where nginx default page is stored
cd /var/share/nginx/html
```

{{% note %}}
6m

* Explain host port mapping
* See nginx processes running inside the container
* docker logs
* docker start
* docker stop
* docker rm -f

{{% /note %}}

---
## Bind Mounts and Volumes

* **Docker Bind Mount**: Mounting host directory/file into a container
* **Docker Volume**: Create a virtual Volume managed by Docker that is mounted into a container

* Volumes are prescribed as "preferred mechanism" for persistence by Docker due to various reasons mentioned [here](https://docs.docker.com/storage/volumes/).
* Bind Mounts are still simpler to manage for single host.

---

### Override the nginx default page with bind mount

```bash
curl -o /tmp/index.html https://pastebin.com/raw/LxcFJW01

docker run --rm -d -p 80:80 -v /tmp/index.html:/var/share/nginx/html/index.html nginx:alpine

# Refresh the page
```

{{% note %}}
6m

{{% /note %}}

---
## Docker Images and Dockerfile

* **Dockerfile** - A template/script that generates Docker Image
* **Docker Image** - Application environment packaged into files
* **Docker Container** - Running application from Docker Image

--

```bash
docker images
```

{{% note %}}
6m

{{% /note %}}

---
## Build our own image

<!-- .slide: class="code" -->

`Dockerfile`
```dockerfile
FROM nginx:alpine

ADD /tmp/index.html /var/share/nginx/html/
```

Commands
```bash
curl -o Dockerfile https://pastebin.com/raw/TfjzNjbY

docker build -t hello-nginx .

# See the image you built listed
docker images

docker run --rm -d -p 80:80 hello-nginx
```

{{% note %}}
6m

{{% /note %}}

---
## Real-World Dockerfile

<!-- .slide: class="code" -->

```dockerfile
FROM node:10-alpine

EXPOSE 3000

ENV NODE_ENV production

RUN mkdir /app
WORKDIR /app
ADD package.json yarn.lock /app/
RUN yarn install --production --pure-lockfile
ADD . /app

CMD ["yarn", "docker:start"]
```

{{% note %}}
6m

* Image Structure. Immutable compressed Layers.


{{% /note %}}

---

## Image Building Best Practices

* One ~~process~~ service per container.
* Keep images stateless.
* Build immutable images for each software release. Don't mount code as volume.
* Keep images as small in size as possible. Add only stuff that you need.
* Minimize docker instructions/layers.
* Utilize multi-stage builds to keep images simple wherever possible.
* Utilize build cache as much as possible.
* Create images to be polymorphic based on environment variables and arguments

{{% note %}}
6m

* One Service per Container: It's okay running nginx and your PHP application (php-fpm) in the same container, but don't cram MySQL or redis in there.
* Minimize layers: Clean up temporary things in the same instruction. Docker layers are immutable (like `git`). Deleting files added in one layer in another layer does not reduce the image size.
* Build Cache: Try to keep the docker instructions sorted in the increasing order of likelyhood of change.

{{% /note %}}

---

{{< slide timing="120" >}}

<div style="font-size: 55px"><b>Okay, it's nice to run a single process inside a container. But, my applications are more complex than that.</b></div>

{{% note %}}
6m

{{% /note %}}

<!-- .slide: class="code" -->

---

## `docker-compose`

* Nice little utility that lets you declare multiple `docker` commands withink a single `yaml` file.
* If `docker` -> `service`, then `docker-compose` -> `application`

{{% note %}}
6m

{{% /note %}}

---

```yaml
version: '3'
services:
  web:
    build: .
    ports:
      - "3000:3000"
  mysql:
    image: "mysql:5.7"
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: my-secret-password
  redis:
    image: "redis:alpine"
```

{{% note %}}
6m

{{% /note %}}

---

## Let's try it!

```bash
curl -o mysql-compose.yml https://pastebin.com/raw/BKudcBhb

docker-compose -f mysql-compose.yml up

# Observe the logs, let mysql server start and visit port 8080 on your IP address

docker-compose -f mysql-compose.yml down
```

{{% note %}}
6m

* What docker-compose does: naming containers, dedicated network
* Always use volume mounts for database servers
* Caution when using docker-compose down
* docker-compose up -d with docker-compose logs
{{% /note %}}

---

## Advantages

* No more "works on my machine". Because of Immutable Code + Environment, literally the same thing runs in all the environments.
* `Dockerfile` serves as documentation of your environment.
* Going back to the older version never breaks. All the dependencies and environment are baked into the image.
* Spinning up the application locally is very easy.
* Forces good application design, and makes automation around deployment and scaling easier.
* Offers isolation from other components from security persepctive.
* Your applications are not tied to a cloud provider/VPS anymore. You can run it anywhere within minutes.

{{% note %}}
6m

{{% /note %}}

---

## Let's run a Laravel Application inside Docker

```bash
git clone https://github.com/deltasquare4/laravel-hello-world

docker-compose up -d

docker stats
```

---
### Thanks!

Slides: [https://github.com/deltasquare4/docker-from-scratch-workshop](https://github.com/deltasquare4/docker-from-scratch-workshop)

Rakshit Menpara ([@deltasquare4](https://twitter.com/deltasquare4))

[improwised.com](https://www.improwised.com)
