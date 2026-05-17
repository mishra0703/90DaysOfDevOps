# Day 30 – Docker Images & Container Lifecycle

## Task
Today's goal is to **understand how images and containers actually work**.

We will:
- Learn the relationship between images and containers
- Understand image layers and caching
- Master the full container lifecycle

---

### Docker Images
1. Pull the `nginx`, `ubuntu`, and `alpine` images from Docker Hub
    - `docker pull nginx ubuntu alpine` : Downloads all three images one by one from docker hub

2. List all images on your machine — note the sizes
    - `docker images` , lists all the images we have on our machine
        
        ```
        IMAGE                ID             DISK USAGE   CONTENT SIZE   EXTRA
        alpine:latest        5b10f432ef3d       13.1MB         3.95MB        
        ubuntu:latest        f3d28607ddd7        160MB         45.3MB    U   
        nginx:latest         06aa3d7be10b        240MB         65.9MB    U   
        hello-world:latest   0e760fdfbc48       25.9kB         9.49kB    U   
        ```   

3. Compare `ubuntu` vs `alpine` — why is one much smaller?
    - Alpine is smaller than Ubuntu , because Ubuntu ships with full GNU libraries, tools, and utilities that we'd expect on a desktop/server. Alpine strips everything down to bare minimum (replaces standard Unix tools with one tiny binary).

4. Inspect an image — what information can you see?
    - Returns a big JSON with everything about the image:
    
    |Field | What it tells you |
    |-----|-------|
    |Id | Full image SHA hash |
    |Os / Architecture | e.g. linux / amd64 |
    |ExposedPorts | Ports the container expects to use |
    |Env | Default environment variables |
    |Cmd / Entrypoint | Default command run on start |
    |Layers | Filesystem layers that make up the image |

5. Remove an image you no longer need
    - `docker rmi nginx`  :  Removes an image by name
    - `docker rmi abc123` :  Removes by image ID
    - `docker rmi nginx --force` : Force remove even if container exists
    - `docker image prune -a` : Remove all unused images at once , but Can't remove an image if a container (even stopped) is still using it , so remove the container first.


---

### Image Layers
1. Run `docker image history nginx` — what do you see?
    - Shows how an image was built — every instruction that was run, in order, with how much space each step took.
    - Bottom to top , that's the actual build order.
        ```
        docker image history ubuntu
        
        IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
        
        f3d28607ddd7   3 weeks ago   umoci raw add-layer --image /home/buildd/roc…   12.3kB    Add rock control     metadata
        <missing>      3 weeks ago   umoci config --image /home/buildd/rockcraft-…   0B        Set annotations
        <missing>      3 weeks ago   umoci config --image /home/buildd/rockcraft-…   0B        Set labels
        <missing>      3 weeks ago   umoci config --image /home/buildd/rockcraft-…   0B        Set default PATH     for     bare-based rock
        <missing>      3 weeks ago   umoci config --image /home/buildd/rockcraft-…   0B        Set default commands
        <missing>      3 weeks ago   umoci config --image /home/buildd/rockcraft-…   0B        Set entrypoint
        <missing>      3 weeks ago   umoci raw add-layer --image /home/buildd/roc…   115MB
        ```

2. Each line is a **layer**. Note how some layers show sizes and some show 0B
    - Some layers are 0B , while some layers have sizes because : 
        - Commands like CMD, EXPOSE, ENV only add metadata — they don't write files to disk, so no size.
        - Commands like RUN, COPY, ADD actually write files — so they have size.

3. Write in your notes: What are layers and why does Docker use them?
    - Each instruction in a Dockerfile creates a read-only layer stacked on top of the previous one. Together they form the final image.
        ```
        ┌──────────────────────┐
        │  FROM debian         │  ← 80MB
        │  RUN apt install     │  ← 58MB
        │  COPY nginx.conf     │  ← 5kB
        │  CMD ["nginx"]       │  ← 0B (metadata)
        └──────────────────────┘
        ```

    - Docker uses layers because : 
        - Caching — If a layer hasn't changed, Docker reuses it. Rebuilds are fast.
        - Sharing — Two images using FROM debian share that layer on disk. No duplication.
        - Efficiency — Only changed layers are pushed/pulled from registry.


---

### Container Lifecycle

1. **Create** a container (without starting it)
    - `docker create --name myapp nginx` : creates but doesn't run

2. **Start** the container
    - `docker start myapp` : Starts the created container

3. **Pause** it and check status
    - `docker pause myapp` : Freezes all processes inside , then `docker ps` : STATUS shows "Up X min (Paused)"

4. **Unpause** it
    - `docker unpause myapp` : Resumes from exact same state

5. **Stop** it
    - `docker stop myapp` : graceful shutdown (SIGTERM → waits → SIGKILL)

6. **Restart** it
    - `docker restart myapp` : Stop + Start in one command

7. **Kill** it
    - `docker kill myapp` : force shutdown instantly (SIGKILL, no waiting)

8. **Remove** it
    - `docker rm myapp` : deletes container (must be stopped first)

*Note*  : `docker stop` is a graceful shutdown that gives the container time to clean up, while `docker kill` is an immediate, forceful termination.

---

### Working with Running Containers

1. Run an Nginx container in detached mode
    - `docker run -d -p 3000:80 nginx`

2. View its **logs**
    - `docker logs container_name/id` 

3. View **real-time logs** (follow mode)
    - `docker logs container_name/id -f`   :  using `-f` terminal enters the follow mode 

4. **Exec** into the container and look around the filesystem
    - `docker exec -it container_name/id bash`  :   Needs to specify shell type 

5. Run a single command inside the container without entering it
    - `docker exec 57ec ls /dev` 
        ```
        Command : docker exec 57ec ls /dev

        Output : core  fd    full  mqueuenull  ptmx  pts   randomshm   stderrstdin stdoutttyurandomzero
        ```

6. **Inspect** the container — find its IP address, port mappings, and mounts
    - `docker inspect container_id`
    - Ip address 
        ```
         "Networks": {
                "bridge": {
                    ...
                    "IPAddress": "172.17.0.3",
                    "MacAddress": "f2:3a:e3:31:75:d6",
                    ...
                }
            }
        ```    
    - Ports : 
        ```
        "Ports": {
                    "80/tcp": [
                        {
                            "HostIp": "0.0.0.0",
                            "HostPort": "3000"
                        },
                        {
                            "HostIp": "::",
                            "HostPort": "3000"
                        }
                    ]
                },
        ```


---

### Cleanup

1. Stop all running containers in one command
    - `docker stop $(docker ps -q)`  :  `-q` --> only return container IDs

2. Remove all stopped containers in one command
    - `docker container prune` : Removes everything that's stopped

3. Remove unused images
    - `docker image prune` : Removes dangling images(layers of Docker data that have lost their name and tag (Orphaned Containers))
    - `docker image prune -a` : Removes ALL unused images

4. Check how much disk space Docker is using
    - `docker system df` : Shows images, containers, volumes size
        ```
        docker system df

        TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
        Images          4         3         1.449GB   13.06MB (0%)
        Containers      3         1         106.5kB   16.38kB (15%)
        Local Volumes   0         0         0B        0B
        Build Cache     12        0         1.436GB   1.423GB
        ```

*Note* : `docker system prune -a`  :  Removes all stopped containers + unused images + networks

---

