+++
title = "docker常用命令"
tags = [ "docker" ]
categories = [ "技术",]
mathjax = false
date = "2024-10-04T09:54:36+08:00"
+++
Docker 是一个开源的容器化平台，允许开发者将应用程序及其依赖项打包到一个轻量级、可移植的容器中。以下是一些常用的 Docker 命令：

### 1. 镜像相关命令

- **`docker pull <image>`**: 从 Docker Hub 或其他镜像仓库拉取镜像。
  ```bash
  docker pull ubuntu
  ```

- **`docker images`**: 列出本地所有的镜像。
  ```bash
  docker images
  ```

- **`docker rmi <image>`**: 删除指定的镜像。
  ```bash
  docker rmi ubuntu
  ```

- **`docker build -t <tag> .`**: 使用 Dockerfile 构建镜像，并指定标签。
  ```bash
  docker build -t myapp:1.0 .
  ```

### 2. 容器相关命令

- **`docker run <image>`**: 运行一个容器。
  ```bash
  docker run -it ubuntu
  ```

- **`docker ps`**: 列出正在运行的容器。
  ```bash
  docker ps
  ```

- **`docker ps -a`**: 列出所有容器，包括已停止的。
  ```bash
  docker ps -a
  ```

- **`docker start <container>`**: 启动一个已停止的容器。
  ```bash
  docker start mycontainer
  ```

- **`docker stop <container>`**: 停止一个正在运行的容器。
  ```bash
  docker stop mycontainer
  ```

- **`docker rm <container>`**: 删除一个容器。
  ```bash
  docker rm mycontainer
  ```

- **`docker exec -it <container> <command>`**: 在运行的容器中执行命令。
  ```bash
  docker exec -it mycontainer bash
  ```

### 3. 网络相关命令

- **`docker network ls`**: 列出所有网络。
  ```bash
  docker network ls
  ```

- **`docker network create <network>`**: 创建一个新的网络。
  ```bash
  docker network create mynetwork
  ```

- **`docker network connect <network> <container>`**: 将容器连接到网络。
  ```bash
  docker network connect mynetwork mycontainer
  ```

### 4. 卷相关命令

- **`docker volume ls`**: 列出所有卷。
  ```bash
  docker volume ls
  ```

- **`docker volume create <volume>`**: 创建一个新的卷。
  ```bash
  docker volume create myvolume
  ```

- **`docker volume rm <volume>`**: 删除一个卷。
  ```bash
  docker volume rm myvolume
  ```

### 5. 其他常用命令

- **`docker logs <container>`**: 查看容器的日志。
  ```bash
  docker logs mycontainer
  ```

- **`docker inspect <container>`**: 查看容器的详细信息。
  ```bash
  docker inspect mycontainer
  ```

- **`docker system prune`**: 清理未使用的数据（包括停止的容器、未使用的网络、未使用的镜像等）。
  ```bash
  docker system prune
  ```

### 6. Docker Compose 命令

- **`docker-compose up`**: 启动 Docker Compose 定义的服务。
  ```bash
  docker-compose up
  ```

- **`docker-compose down`**: 停止并删除 Docker Compose 定义的服务。
  ```bash
  docker-compose down
  ```

- **`docker-compose ps`**: 列出 Docker Compose 定义的服务。
  ```bash
  docker-compose ps
  ```
