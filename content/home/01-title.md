+++
title = "Docker From Scratch"
outputs = ["Reveal"]
+++

# Docker From Scratch

Hands-on Workshop

{{% note %}}
<pre>
1m

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
## What is Docker?

* A cloud infrastructure company (formerly dotCloud) that created Docker as an internal tool
* Container Runtime daemon that runs and manages Containers on top of Linux Kernel
* CLI client to the Docker Daemon
* Container Image Packaging Format, now evolved into OCI Image

{{% note %}}
3m

{{% /note %}}

---
## Containers vs Virtual Machines

![Custom Autoscaler](https://i0.wp.com/blog.docker.com/wp-content/uploads/Blog.-Are-containers-..VM-Image-1.png)

(Source: [Docker Blog: Are Containers Replacing Virtual Machines? by Jenny Fong](https://blog.docker.com/2018/08/containers-replacing-virtual-machines/))

{{% note %}}
4m

{{% /note %}}

---
## History of Containers

* **1979**: Development of `chroot` in Unix V7
* **2003**: Google Created its internal cluster management system Borg
* **2004**: Development Process Containers, which eventually became `cgroups`
* **2008**: `cgroups` got absorbed in Linux Kernel in 2008, which led to development of LXC
* **2013**: Docker was built on top of LXC
* **2015**: Open Container Initiative (OCI) was created by Docker, CoreOS and others to standardize containers. Docker donated its container runtime `runc` to the OCI. Kubernetes was born.
* **2017 Onwards**: Kubernetes emerged as a victor in Container Orchestrator Wars™. Seeds of alternative ecosystem were sewn with development of alternative tools like CRI-O, and Podman. Evolution is still ongoing.

{{% note %}}
5m


{{% /note %}}

---
## Enough Talk. Let's Get Our Hands Dirty!

* Visit <app-link>, and authenticate yourself through the gmail account you used to register for the workshop

{{% note %}}
6m

{{% /note %}}

---
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
6m

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

# Clean up
docker stop <nginx-container-name>
docker rm <nginx-container-name>
```

{{% note %}}
6m

* Explain host port mapping
* See nginx processes running inside the container

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

* Keep images stateless.
* Build immutable images for each software release. Don't mount code as volume.
* Keep images as small in size as possible. Add only stuff that you need.
* Minimize docker instructions/layers.
* Utilize multi-stage builds to keep images simple wherever possible.
* Utilize build cache as much as possible.
* Create images to be polymorphic based on environment variables and arguments

{{% note %}}
6m

* Minimize layers: Clean up temporary things in the same instruction. Docker layers are immutable (like `git`). Deleting files added in one layer in another layer does not reduce the image size.
* Build Cache: Try to keep the docker instructions sorted in the increasing order of likelyhood of change.

{{% /note %}}

---

## Advantages

* Immutable Code + Environment.

{{% note %}}
6m

{{% /note %}}

---
### Thanks!

Slides: [https://github.com/deltasquare4/docker-from-scratch-workshop](https://github.com/deltasquare4/docker-from-scratch-workshop)

Rakshit Menpara ([@deltasquare4](https://twitter.com/deltasquare4))

[improwised.com](https://www.improwised.com)
