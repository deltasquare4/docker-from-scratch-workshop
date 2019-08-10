+++
title = "Docker From Scratch"
outputs = ["Reveal"]
+++

{{< slide timing="180" >}}

# Docker From Scratch

Hands-on Workshop

Slides: [http://bit.ly/docker-workshop-rj](http://bit.ly/docker-workshop-rj)

{{% note %}}
<pre>
* Who am I?
* How many of you have used
  Docker?
* In production?

* Workshop Ground Rules:
  * Open slides locally to copy commands
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

**Linux/Mac:**

```
curl -o id_rsa https://pastebin.com/raw/rp9NVugW
curl -o id_rsa.pub https://pastebin.com/raw/VZS2MkuT
ssh-add id_rsa
ssh root@<your-machine-ip>
```

**Windows:**

Download [https://pastebin.com/raw/EAMDxdZT](https://pastebin.com/raw/EAMDxdZT) and use it with PuTTY

{{% note %}}
20m

{{% /note %}}

---

{{< slide timing="240" >}}

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

{{< slide timing="180" >}}

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
<pre>
27m

* docker run --rm
* ps
</pre>
{{% /note %}}

---

{{< slide timing="300" >}}

## Run a background container: `nginx`

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
cd /usr/share/nginx/html
```

{{% note %}}
<pre>
* Explain host port mapping
* See nginx processes running inside the container
* host port mappings
* docker logs -f
* docker start
* docker stop
* docker rm -f
</pre>
{{% /note %}}

---

{{< slide timing="240" >}}

## Bind Mounts and Volumes

* **Docker Bind Mount**: Mounting host directory/file into a container
* **Docker Volume**: Create a virtual Volume managed by Docker that is mounted into a container

* Volumes are prescribed as "preferred mechanism" for persistence by Docker due to various reasons mentioned [here](https://docs.docker.com/storage/volumes/).
* Bind Mounts are still simpler to manage for single host.

{{% note %}}
<pre>
36m

* Docker Volumes
* How to ensure data persistence: Separate EBS/Block Storage
* Bind Mount Pattern: Share single mount with multiple containers
</pre>
{{% /note %}}

---

{{< slide timing="180" >}}

### Override the nginx default page with bind mount

```bash
curl -o /tmp/index.html https://pastebin.com/raw/k8DyhGqf

docker run --rm -p 80:80 -v /tmp/index.html:/usr/share/nginx/html/index.html nginx:alpine

# Refresh the page
```

{{% note %}}
<pre>
* Docker Volume mappings
</pre>
{{% /note %}}

---

{{< slide timing="180" >}}

## Docker Images and Dockerfile

* **Dockerfile** - A template/script that generates Docker Image
* **Docker Image** - Application environment packaged into files
* **Docker Container** - Running application from Docker Image

--

```bash
docker images
```

{{% note %}}
<pre>
42m
</pre>
{{% /note %}}

---

{{< slide timing="240" >}}

## Build our own image

<!-- .slide: class="code" -->

`Dockerfile`
```dockerfile
FROM nginx:alpine

ADD /tmp/index.html /usr/share/nginx/html/
```

Commands
```bash
curl -o Dockerfile https://pastebin.com/raw/WDznhx8n

docker build -t hello-nginx .

# See the image you built listed
docker images

docker run --rm -p 80:80 hello-nginx
```

{{% note %}}
<pre>
* How to decide what to run at build-time and what to run at runtime.
* Base images, and role of layers
</pre>
{{% /note %}}

---

{{< slide timing="300" >}}

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
<pre>
51m

* Image Structure. Immutable compressed Layers.
* FROM
* RUN
* ADD/COPY
* ENTRYPOINT/CMD
* ENV/EXPOSE/ARG
* WORKDIR
* VOLUME/USER/HEALTHCHECK
* Build Caching

</pre>
{{% /note %}}

---

{{< slide timing="240" >}}

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
<pre>
* One Service per Container: It's okay running nginx
  and your PHP application (php-fpm) in the same container,
  but don't cram MySQL or redis in there.
* Minimize layers: Clean up temporary things in the same
  instruction. Docker layers are immutable (like `git`).
  Deleting files added in one layer in another layer does
  not reduce the image size.
* Build Cache: Try to keep the docker instructions sorted
  in the increasing order of likelyhood of change.
</pre>
{{% /note %}}

---

{{< slide timing="60" >}}

<div style="font-size: 55px"><b>Okay, it's nice to run a single process inside a container. But, my applications are more complex than that.</b></div>

{{% note %}}
56m
{{% /note %}}

<!-- .slide: class="code" -->

---

{{< slide timing="120" >}}

## `docker-compose`

* Nice little utility that lets you declare multiple `docker` commands withink a single `yaml` file.
* If `docker` -> `service`, then `docker-compose` -> `application`

{{% note %}}
58m
{{% /note %}}

---

{{< slide timing="360" >}}

```yaml
version: '3'
services:
  web:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - mysql
      - redis
  mysql:
    image: "mysql:5.7"
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: my-secret-password
  redis:
    image: "redis:alpine"
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 50M
        reservations:
          cpus: '0.25'
          memory: 20M
```

{{% note %}}
<pre>
64m

* version/services
* build/image
* restart
* environment/env_file/ports
* depends_on/link
* volumes/networks
</pre>
{{% /note %}}

---

{{< slide timing="300" >}}

## Let's try it!

```bash
curl -o mysql-compose.yml https://pastebin.com/raw/BKudcBhb

docker-compose -f mysql-compose.yml up

# Observe the logs, let mysql server start and visit port 8080 on your IP address

docker-compose -f mysql-compose.yml down
```

{{% note %}}
<pre>
* What docker-compose does: naming containers, dedicated network
* Always use volume mounts for database servers
* Caution when using docker-compose down
* docker-compose up -d with docker-compose logs
* docker-compose ps/start/stop/top/scale
</pre>
{{% /note %}}

---

{{< slide timing="300" >}}

## Let's run a Laravel Application inside Docker

```bash
git clone https://github.com/deltasquare4/laravel-hello-world

docker-compose up -d

docker stats
```

{{% note %}}
<pre>
74m

* structure of the application
* dockerfile/docker-compose.yml
* database link
</pre>
{{% /note %}}

---

{{< slide timing="180" >}}

## Advantages

* No more "works on my machine". Because of Immutable Code + Environment, literally the same thing runs in all the environments.
* `Dockerfile` serves as documentation of your environment.
* Going back to the older version never breaks. All the dependencies and environment are baked into the image.
* Spinning up the application locally is very easy.
* Forces good application design, and makes automation around deployment and scaling easier.
* Offers isolation from other components from security persepctive.
* Your applications are not tied to a cloud provider/VPS anymore. You can run it anywhere within minutes.

{{% note %}}
77m
{{% /note %}}

---

{{< slide timing="180" >}}

## Drawbacks

* Docker daemon runs as root. Can be a potential security risk.
* All containers run as child processes of Docker daemon. Bringing down the daemon will bring down all the containers.
* Persistent data is complicated. Servers cannot be treated as throwaway completely in case of stateful load.
* Docker does much more than container management; is bloated as a result.

{{% note %}}
<pre>
80m

* Persistence: Third-party tools - ceph/glusterfs/nfs/portainer
* Bloated: List of commands, swarm
</pre>
{{% /note %}}

---

{{< slide timing="120" >}}

### Thanks!

Feedback: [https://forms.gle/xMUpyhZPPxBnMTU19](https://forms.gle/xMUpyhZPPxBnMTU19)

Slides: [https://github.com/deltasquare4/docker-from-scratch-workshop](https://github.com/deltasquare4/docker-from-scratch-workshop)
Laravel App Repo: [https://github.com/deltasquare4/laravel-hello-world](https://github.com/deltasquare4/laravel-hello-world)
Base Image Repo: [https://github.com/deltasquare4/docker-php-base](https://github.com/deltasquare4/docker-php-base)

Rakshit Menpara ([@deltasquare4](https://twitter.com/deltasquare4))

[improwised.com](https://www.improwised.com)

{{% note %}}
<pre>
82m
</pre>
{{% /note %}}

---

{{< slide timing="420" >}}

### Bonus 1: Registry

Create a Docker ID Here: [https://hub.docker.com/signup](https://hub.docker.com/signup)

```bash
docker login

docker tag <image>:<tag> <image>:<new-tag>

docker push <image>:<new-tag>
```

{{% note %}}
<pre>
89m
</pre>
{{% /note %}}

---

{{< slide timing="600" >}}

### Bonus 2: Portainer

Run Portainer with instructions [here](https://www.portainer.io/installation/)

{{% note %}}
<pre>
99m
* Insecure by default. Use with nginx in front with basic authentication, or buy their auth plugin.
</pre>
{{% /note %}}

---

{{< slide timing="180" >}}

### Bonus 3: Buildah + Skopeo + Podman

These next-generation Open Source tools are designed to replace Docker, and address many of the disadvantages.

{{% note %}}
<pre>
99m

* Podman: Command-compatible with Docker CLI.
* Podman: No daemon.
* Buildah: Rootless containers.
* Skopeo: Secure handling of Images and Remote
  Registries. Supports image signing etc.
</pre>
{{% /note %}}
