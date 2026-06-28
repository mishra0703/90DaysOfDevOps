# Docker Cheat Sheet

## Container Commands
| Command | Description |
|---|---|
| `docker run -d -p 8080:80 --name web nginx` | Run container, detached, port-mapped, named |
| `docker run -it ubuntu bash` | Run interactively with a shell |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers (incl. stopped) |
| `docker stop <container>` | Stop a running container gracefully |
| `docker rm <container>` | Remove a stopped container |
| `docker rm -f <container>` | Force remove (kills if running) |
| `docker exec -it <container> bash` | Open a shell inside a running container |
| `docker logs -f <container>` | Stream container logs (follow mode) |
| `docker logs --tail 100 <container>` | Show last 100 log lines |
| `docker inspect <container>` | Full JSON metadata (IP, mounts, env, etc.) |
| `docker restart <container>` | Restart a container |
| `docker cp <container>:/path ./local` | Copy file out of a container |

## Image Commands
| Command | Description |
|---|---|
| `docker build -t myapp:1.0 .` | Build image from Dockerfile in current dir |
| `docker build -t myapp:1.0 -f Dockerfile.prod .` | Build using a specific Dockerfile |
| `docker pull <image>:<tag>` | Download image from registry |
| `docker push <user>/<image>:<tag>` | Push image to registry (Docker Hub etc.) |
| `docker tag myapp:1.0 user/myapp:latest` | Tag an image for a registry |
| `docker images` / `docker image ls` | List local images |
| `docker rmi <image>` | Remove an image |
| `docker history <image>` | Show image layer history |

## Volume Commands
| Command | Description |
|---|---|
| `docker volume create mydata` | Create a named volume |
| `docker volume ls` | List all volumes |
| `docker volume inspect mydata` | Show mount point, driver, scope |
| `docker volume rm mydata` | Remove a volume |
| `docker run -v mydata:/data ...` | Mount named volume into container |
| `docker run -v $(pwd):/app ...` | Bind mount host dir into container |

## Network Commands
| Command | Description |
|---|---|
| `docker network create mynet` | Create a user-defined bridge network |
| `docker network ls` | List networks |
| `docker network inspect mynet` | Show connected containers, subnet, gateway |
| `docker network connect mynet <container>` | Attach running container to a network |
| `docker network disconnect mynet <container>` | Detach container from a network |
| `docker network rm mynet` | Remove a network |

## Compose Commands
| Command | Description |
|---|---|
| `docker compose up -d` | Start all services, detached |
| `docker compose up -d --build` | Rebuild images then start |
| `docker compose down` | Stop and remove containers, networks |
| `docker compose down -v` | Also remove named volumes |
| `docker compose ps` | List services and their status |
| `docker compose logs -f <service>` | Follow logs for one service |
| `docker compose build` | Build/rebuild service images |
| `docker compose restart <service>` | Restart a single service |
| `docker compose exec <service> sh` | Shell into a running service |

## Cleanup Commands
| Command | Description |
|---|---|
| `docker system df` | Disk usage by images/containers/volumes |
| `docker system prune` | Remove unused containers, networks, dangling images |
| `docker system prune -a` | Also remove all unused images, not just dangling |
| `docker container prune` | Remove all stopped containers |
| `docker image prune -a` | Remove all unused (not just dangling) images |
| `docker volume prune` | Remove unused volumes |
| `docker network prune` | Remove unused networks |

## Dockerfile Instructions
| Instruction | Description |
|---|---|
| `FROM node:20-alpine` | Base image to build from |
| `RUN npm install` | Execute command at build time, creates a layer |
| `COPY . .` | Copy files from host into image |
| `ADD` | Like COPY, but also handles URLs/tar extraction (prefer COPY) |
| `WORKDIR /app` | Set working directory for subsequent instructions |
| `EXPOSE 3000` | Document the port the app listens on (no actual mapping) |
| `ENV NODE_ENV=production` | Set environment variable in the image |
| `ARG BUILD_VERSION` | Build-time-only variable (not in final image) |
| `CMD ["node", "server.js"]` | Default command, overridable at `docker run` |
| `ENTRYPOINT ["node"]` | Fixed executable; CMD becomes its args |
| `HEALTHCHECK CMD curl -f http://localhost/ || exit 1` | Define container health check |
| `USER node` | Run as non-root user |
| `VOLUME /data` | Declare a mount point |
