---
author: "Selwyn Rogers"
title: "Run GitLab Runners in Podman Containers with Docker Executors"
date: "2020-08-27"
description: "Currently, there is no official GitLab Runner Executor for Podman. This presents a challenge for users of RedHat Enterprise Linux 8, which has replaced Docker with Podman. However, I have found a simple solution to run Runners with Docker Executors on RHEL8 without installing Docker natively or having to write a custom executor from scratch. The trick is to run a Podman container that runs Docker inside and shares its Docker Socket as a volume between containers."
tags: ["openshift", "redhat","vault"]
ShowToc: false
ShowBreadCrumbs: false
---

Download my example here: https://github.com/selloween/podman-gitlab-runner-docker-executor

## Preface
Currently, there is no official GitLab Runner Executor for Podman. As RedHat Enterprise Linux 8 has replaced Docker with Podman, I had to find a simple solution to run Runners with Docker Executors on RHEL8 without installing Docker natively or writing a custom executor from scratch. The trick is to run a Podman Container that runs Docker inside and shares its Docker Socket as a volume between containers!

## Creating directory structure
I prefer to create a podman directory in `/opt` for all container bind mounts.
```bash
mkdir -p /opt/podman/gitlab-runner
mkdir -p /opt/podman/gitlab-runner/certs
mkdir -p /opt/podman/dind/docker
touch /opt/podman/gitlab-runner/config.toml
```

## Docker in Podman Container
First, create a Podman container using the official `dind` ("Docker in Docker") Image from Dockerhub. For this, I created a simple bash script located in the `/dind` directory. It's important that the "Docker in Podman" container is started before starting the GitLab Runner container.

#### Mounting and sharing Docker Socket
It's important to mount `/var/run` as a volume so that the GitLab Runner container can access the Docker Socket. Also, you will want to bind mount `/etc/docker` so that you can easily modify configuration files later on. (see the bash script below)
```bash
#!/bin/bash
podman run -d \
        --restart=always \
        --name dind \
        -v docker_run:/var/run \
        -v /opt/podman/dind/docker:/etc/docker \
        docker:dind
```
**Create the container:**
```bash
chmod +x ./dind/podman.sh
./dind/podman.sh
``` 

## GitLab Runner Container
### Docker Host
To ensure proper communication between the GitLab Runner and the Docker host, it's essential to set the `DOCKER_HOST` environment variable to the IP of your Podman host. As both containers should be running on the same host, you can use the local IP `127.0.0.1`.

### Docker Socket
To enable the GitLab Runner to connect to the Docker Socket and utilize the Docker Executor, you need to "import" the `/var/run` volume from the "Docker in Podman" container using `--volumes-from dind`. This makes the Docker Socket available to the GitLab Runner container.

### TLS
To ensure successful connection of your Runner to the GitLab Server, you need to add your TLS certificates (key, cert, and CA cert) to `/opt/podman/gitlab-runner/certs`.
```
├── certs
│   ├── ca.crt
│   ├── your_host.crt
│   ├── your_host.key
│   └── your_host.pem
```



### Starting the GitLab Runner Container
```bash
#!/bin/bash
podman run -d \
        --restart=always \
        --name gitlab-runner \
        -e DOCKER_HOST=127.0.0.1 \
        -v /opt/podman/gitlab-runner/certs:/etc/gitlab-runner/certs:ro \
        -v /opt/podman/gitlab-runner/config.toml:/etc/gitlab-runner/config.toml \
        --volumes-from dind \
        gitlab/gitlab-runner:latest
```
**Create and start the container:**
```bash
chmod +x ./gitlab-runner/podman.sh
./gitlab-runner/podman.sh
```

### Create and register Runner
After starting the container, you need to create and register a Runner to your GitLab Server by using the GitLab Runner command line interface and following the prompt wizard. When prompted, make sure to select the `Docker Executor` and set the default image to `docker:stable`.
```bash
podman exec -it gitlab-runner gitlab-runner 
```

### Edit Runner Config
After registering your Runner, edit the updated runner configuration located at `/opt/podman/gitlab-runner/config.toml`. It's important to add `/var/run/docker.sock:/var/run/docker.sock` to the volumes array: `volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache"]`.

Additionally, set the NO_PROXY environment variables: environment = ["NO_PROXY=docker:2375,docker:2376", "no_proxy=docker:2375,docker:2376"]. This will pass the Docker socket to the Docker container that gets spawned during a Build Process. You are now running Docker in Podman running Docker!

#### config.toml
Your `config.toml` should look like bellow.
If you are using internal certificates, set `tls_verify` to false. This will disable certificate verification and allow the GitLab Runner to connect to the GitLab Server using internal certificates.

```toml
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "docker"
  url = "https://my-gitlab-server/"
  token = "XXXXXXXXXXXXXXXX"
  executor = "docker"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
  [runners.docker]
    # in case you are using internal certificates set tls_verify to false
    tls_verify = false
    image = "docker:stable"
    privileged = true
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache"]
    environment = ["NO_PROXY=docker:2375,docker:2376", "no_proxy=docker:2375,docker:2376"]
    dns = []
    extra_hosts = []
    shm_size = 0
```