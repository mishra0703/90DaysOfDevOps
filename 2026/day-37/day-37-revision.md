## Self-Assessment Checklist

- [ can do ] Run a container from Docker Hub [ interactive(-it) + detached(-d) ]
- [ can do ] List, stop, remove containers and images
- [ can do ] Explain image layers and how caching works
- [ can do ] Write a Dockerfile from scratch with FROM, RUN, COPY, WORKDIR, CMD
- [ can do ] Explain CMD vs ENTRYPOINT
- [ can do ] Build and tag a custom image
- [ can do ] Create and use named volumes
- [ can do ] Use bind mounts
- [ can do ] Create custom networks and connect containers
- [ can do ] Write a docker-compose.yml for a multi-container app
- [ can do ] Use environment variables and .env files in Compose
- [ shaky ] Write a multi-stage Dockerfile
- [ can do ] Push an image to Docker Hub
- [ can do ] Use healthchecks and depends_on

---

## Quick-Fire Questions

### 1. What is the difference between an image and a container?
- An image is the read-only template (filesystem + config) built from a Dockerfile. A container is a running (or stopped) instance of that image, with its own writable layer on top.

### 2. What happens to data inside a container when you remove it?
- Anything written to the container's writable layer is gone permanently. Only data in a mounted volume or bind mount survives, since that lives outside the container's lifecycle.

### 3. How do two containers on the same custom network communicate?
- Docker's embedded DNS lets them reach each other by container/service name (e.g. mongo:27017) instead of IP. This only works on user-defined networks, not the default bridge.

### 4. What does `docker compose down -v` do differently from `docker compose down`?
- Both stop and remove containers and the network. `-v` additionally removes named volumes, so anything in your DB/data volume is wiped too.

### 5. Why are multi-stage builds useful?
- They let you use heavy build tools (compilers, dev deps) in early stages, then ship only the final compiled output in a slim runtime stage — smaller image, smaller attack surface.

### 6. What is the difference between `COPY` and `ADD`?
- `COPY` just copies local files/dirs into the image. `ADD` does that plus auto-extracts tar archives and can fetch from URLs. Best practice is to use COPY unless you specifically need ADD's extra behavior.

### 7. What does `-p 8080:80` mean?
- Maps host port 8080 to container port 80 (host:container). Traffic hitting localhost:8080 on the host gets forwarded to port 80 inside the container.

### 8. How do you check how much disk space Docker is using?
- `docker system df` shows space used by images, containers, volumes, and build cache, with a breakdown of reclaimable space.

---

