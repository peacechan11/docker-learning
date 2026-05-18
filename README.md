# Docker Learning

This repository is a beginner-friendly Docker learning project that covers Docker fundamentals, hands-on labs, Dockerfile usage, custom image builds, volumes, networks, Docker Compose, production practices, and Docker Swarm basics.

The goal of this repository is to help learners understand Docker step by step through notes, examples, and practical tasks.

---

## 📌 Repository Overview

This repository is designed for learning Docker through both theory and hands-on practice.

Main topics include:

- Docker basic commands
- Docker images and containers
- Dockerfile and custom image builds
- Docker volumes
- Docker networks
- Docker Compose
- Multi-container applications
- Advanced production practices
- Docker Swarm basics
- Service scaling and orchestration

---

## 📁 Project Structure

```text
docker-learning/
├── compose-app/
├── python-docker-app/
├── advanced_production_practices_notes.md
├── docker-swarm-lab.md
├── docker_advanced_commands_notes.md
├── docker_basic_commands_notes.md
├── docker_network_volume_compose_notes.md
├── docker_swarm_notes.md
└── dockerfile_build_notes.md
```

---

## 📚 Contents

| File / Folder | Description |
|---|---|
| `compose-app/` | Example project for running a multi-container application with Docker Compose |
| `python-docker-app/` | Sample Python application for Docker image build and container run practice |
| `docker_basic_commands_notes.md` | Notes for basic Docker commands |
| `dockerfile_build_notes.md` | Notes about writing Dockerfiles and building custom images |
| `docker_network_volume_compose_notes.md` | Notes about Docker networks, volumes, and Compose |
| `docker_advanced_commands_notes.md` | Notes for advanced Docker commands |
| `advanced_production_practices_notes.md` | Notes for production-ready Docker practices |
| `docker_swarm_notes.md` | Explanation of Docker Swarm concepts |
| `docker-swarm-lab.md` | Hands-on Docker Swarm lab |

---

## 🚀 Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/peacechan11/docker-learning.git
cd docker-learning
```

---

### 2. Check Docker Installation

```bash
docker --version
```

Check Docker Compose:

```bash
docker compose version
```

---

### 3. Run Basic Docker Commands

Pull and run an Nginx container:

```bash
docker pull nginx:alpine
docker run -d --name web -p 8080:80 nginx:alpine
```

Open the application in a browser:

```text
http://localhost:8080
```

Stop and remove the container:

```bash
docker stop web
docker rm web
```

---

## 🐳 Dockerfile Example

Example Dockerfile for serving a custom HTML page with Nginx:

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

Build the image:

```bash
docker build -t myapp:v1 .
```

Run the container:

```bash
docker run -d --name myapp -p 8080:80 myapp:v1
```

Test in the browser:

```text
http://localhost:8080
```

---

## 📦 Docker Volume Example

Create a named volume:

```bash
docker volume create labdata
```

Run a container and mount the volume:

```bash
docker run -d --name datastore -v labdata:/data alpine sleep infinity
```

List Docker volumes:

```bash
docker volume ls
```

Inspect the volume:

```bash
docker volume inspect labdata
```

---

## 🌐 Docker Network Example

Create a user-defined bridge network:

```bash
docker network create labnet
```

Run two containers on the same network:

```bash
docker run -d --name app1 --network labnet alpine sleep infinity
docker run -d --name app2 --network labnet alpine sleep infinity
```

Test communication by container name:

```bash
docker exec app1 ping -c 4 app2
```

---

## 🧩 Docker Compose

Docker Compose is used to define and run multi-container applications.

Start services:

```bash
docker compose up -d
```

Check running services:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs
```

Stop and remove services:

```bash
docker compose down
```

---

## 🏭 Advanced Production Practices

For production-ready Docker images and containers, follow these practices:

- Use multi-stage builds
- Use small base images
- Run containers as a non-root user
- Set CPU and memory limits
- Use `.dockerignore`
- Never bake secrets into images
- Scan images for vulnerabilities
- Add `HEALTHCHECK`
- Use restart policies
- Use Docker Buildx for multi-platform builds

---

## 🔐 Example `.dockerignore`

```dockerignore
.env
.git
node_modules
*.pem
secrets/
```

A `.dockerignore` file prevents unnecessary or sensitive files from being sent to the Docker build context.

---

## 🩺 Healthcheck Example

```dockerfile
HEALTHCHECK --interval=30s \
  --timeout=5s \
  --retries=3 \
  CMD curl --fail http://localhost:3000 || exit 1
```

A healthcheck helps Docker know whether the application inside the container is working correctly.

---

## 🐝 Docker Swarm

Docker Swarm is Docker’s native orchestration tool. It allows multiple Docker hosts to work together as a cluster and run services across multiple nodes.

Initialize a Swarm cluster:

```bash
docker swarm init --advertise-addr <MANAGER-IP>
```

Create a service:

```bash
docker service create \
  --name my-web \
  --replicas 3 \
  --publish 8080:80 \
  nginx:alpine
```

Scale the service:

```bash
docker service scale my-web=5
```

List services:

```bash
docker service ls
```

View service tasks:

```bash
docker service ps my-web
```

Remove the service:

```bash
docker service rm my-web
```

---

## 🧪 Practice Tasks

You can use this repository to practice the following Docker tasks.

### Task 1: Run Your First Container

```bash
docker run -d --name nginx-lab -p 8080:80 nginx:alpine
```

---

### Task 2: Build a Custom Image

```bash
docker build -t myapp:v1 .
```

---

### Task 3: Create a Named Volume

```bash
docker volume create labdata
docker run -d --name datastore -v labdata:/data alpine sleep infinity
```

---

### Task 4: Create a Custom Network

```bash
docker network create labnet
docker run -d --name app1 --network labnet alpine sleep infinity
docker run -d --name app2 --network labnet alpine sleep infinity
docker exec app1 ping -c 4 app2
```

---

### Task 5: Deploy a Swarm Service

```bash
docker swarm init
docker service create --name my-web --replicas 3 --publish 8080:80 nginx:alpine
```

---

## ✅ Suggested Learning Path

Recommended order:

1. Read `docker_basic_commands_notes.md`
2. Practice basic container commands
3. Read `dockerfile_build_notes.md`
4. Build custom Docker images
5. Read `docker_network_volume_compose_notes.md`
6. Practice volumes, networks, and Docker Compose
7. Read `docker_advanced_commands_notes.md`
8. Study `advanced_production_practices_notes.md`
9. Read `docker_swarm_notes.md`
10. Practice `docker-swarm-lab.md`

---

## 🛠️ Useful Docker Commands

```bash
docker ps
docker ps -a
docker images
docker pull <image>
docker run <image>
docker stop <container>
docker rm <container>
docker rmi <image>
docker logs <container>
docker exec -it <container> sh
docker inspect <container>
docker network ls
docker volume ls
```

---

## 🧹 Cleanup Commands

Remove stopped containers:

```bash
docker container prune
```

Remove unused images:

```bash
docker image prune
```

Remove unused volumes:

```bash
docker volume prune
```

Remove unused networks:

```bash
docker network prune
```

Remove all unused Docker resources:

```bash
docker system prune
```

---

## 🎯 Learning Goals

After completing this repository, you should be able to:

- Run Docker containers
- Manage Docker images
- Write Dockerfiles
- Build custom Docker images
- Use Docker volumes for persistent data
- Use Docker networks for container communication
- Run multi-container applications with Docker Compose
- Apply production-ready Docker practices
- Understand Docker Swarm basics
- Scale and manage services with Docker Swarm

---

## 👤 Author

Created by [peacechan11](https://github.com/peacechan11)

---

## 📄 License

This repository is for learning and practice purposes.
