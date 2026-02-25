# Docker Commands Cheat Sheet

## Running Containers

| Task | Command |
|------|----------|
| Run container | `docker run nginx` |
| Run detached | `docker run -d nginx` |
| Run with name | `docker run --name my-nginx -d nginx` |
| Port mapping | `docker run -p 8080:80 nginx` |
| Environment variable | `docker run -e NODE_ENV=production my-app` |
| Multiple env vars | `docker run -e DB_HOST=localhost -e DB_PORT=5432 my-app` |
| Bind mount volume | `docker run -v $(pwd):/app my-app` |
| Named volume | `docker run -v my-volume:/data my-app` |
| Interactive shell | `docker run -it ubuntu bash` |
| Auto-remove on stop | `docker run --rm nginx` |
| Set restart policy | `docker run --restart=always nginx` |
| Limit memory | `docker run -m 512m nginx` |
| Limit CPU | `docker run --cpus="1.5" nginx` |

---

##  Listing Containers

| Task | Command |
|------|----------|
| Show running | `docker ps` |
| Show all (incl. stopped) | `docker ps -a` |
| Show only IDs | `docker ps -q` |

---

##  Stopping & Starting

| Task | Command |
|------|----------|
| Stop container | `docker stop container_id` |
| Stop multiple | `docker stop c1 c2` |
| Stop all running | `docker stop $(docker ps -q)` |
| Start container | `docker start container_id` |
| Restart container | `docker restart container_id` |
| Pause container | `docker pause container_id` |
| Unpause container | `docker unpause container_id` |
| Kill immediately | `docker kill container_id` |

---

##  Removing Containers

| Task | Command |
|------|----------|
| Remove container | `docker rm container_id` |
| Force remove running | `docker rm -f container_id` |
| Remove stopped containers | `docker container prune` |
| Remove all containers | `docker rm $(docker ps -aq)` |

---

##  Inspection & Logs

| Task | Command |
|------|----------|
| View logs | `docker logs container_id` |
| Follow logs | `docker logs -f container_id` |
| Last 100 lines | `docker logs --tail 100 container_id` |
| Inspect details | `docker inspect container_id` |
| Live resource usage | `docker stats` |
| Running processes | `docker top container_id` |
| Filesystem changes | `docker diff container_id` |
| Port bindings | `docker port container_id` |

---

## Execute Inside Container

| Task | Command |
|------|----------|
| Open bash shell | `docker exec -it container_id bash` |
| Open sh shell | `docker exec -it container_id sh` |
| Run single command | `docker exec container_id ls -la /app` |

---

##  Networking 

| Task | Command |
|------|----------|
| Run on network | `docker run --network my-network nginx` |
| Connect running container | `docker network connect my-network container_id` |

---

##  File Copy

| Task | Command |
|------|----------|
| Copy from container → host | `docker cp container_id:/app/file.txt .` |
| Copy from host → container | `docker cp file.txt container_id:/app/` |

---

##  Cleanup Power Commands

| Task | Command |
|------|----------|
| Remove stopped containers | `docker container prune` |
| Stop + remove all | `docker stop $(docker ps -q) && docker rm $(docker ps -aq)` |

---


## Docker Images

| Task | Command |
|------|----------|
| List images | `docker images` or `docker image ls` |
| Pull image | `docker pull nginx` |
| Build image | `docker build -t my-app:latest .` |
| Tag image | `docker tag my-app:latest myrepo/my-app:v1.0` |
| Remove image | `docker rmi image_id` |
| Remove unused images | `docker image prune` |
| Remove all unused images | `docker image prune -a` |
| Remove all images | `docker rmi $(docker images -q)` |
| Inspect image | `docker image inspect image_id` |
| Push image to registry | `docker push myrepo/my-app:v1.0` |


---


## Docker Volumes

| Task | Command |
|------|----------|
| List volumes | `docker volume ls` |
| Create volume | `docker volume create my-volume` |
| Inspect volume | `docker volume inspect my-volume` |
| Remove volume | `docker volume rm my-volume` |
| Remove unused volumes | `docker volume prune` |
| Remove all unused volumes | `docker volume prune -a` |

---

## Docker Networking

### Network Management

| Task | Command |
|------|----------|
| List networks | `docker network ls` |
| Create network | `docker network create my-network` |
| Create bridge network | `docker network create --driver bridge my-network` |
| Inspect network | `docker network inspect my-network` |
| Remove network | `docker network rm my-network` |
| Remove unused networks | `docker network prune` |

---

### Connect Containers to Network

| Task | Command |
|------|----------|
| Run container on network | `docker run --network my-network nginx` |


---


## Docker Compose

| Task | Command |
|------|----------|
| Start services | `docker compose up` |
| Start in detached mode | `docker compose up -d` |
| Stop services | `docker compose stop` |
| Remove containers & networks | `docker compose down` |
| Rebuild images | `docker compose up --build` |
| List running services | `docker compose ps` |
| View logs (all services) | `docker compose logs` |
| View logs (specific service) | `docker compose logs service_name` |
| Restart services | `docker compose restart` |


---