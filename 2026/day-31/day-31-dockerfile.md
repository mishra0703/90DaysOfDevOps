# Day 31 – Dockerfile: Build Your Own Images

## Task
Our today's goal is to **write Dockerfiles and build custom images**.

This is the skill that separates someone who uses Docker from someone who actually ships with Docker.

---

## Our First Dockerfile

1. Create a folder called `my-first-image`
    

2. Inside it, create a `Dockerfile` that:
   - Uses `ubuntu` as the base image
   - Installs `curl`
   - Sets a default command to print `"Hello from my custom image!"`

3. Build the image and tag it `my-ubuntu:v1`

4. Run a container from your image

   ```bash
   Steps taken :  
   
   #1  : Created folder
   mkdir my-first-image
   cd my-first-image
   
   #2  : Created Docker file
   vim Dockerfile
   
   #3  : Wrote script in Dockerfile
   FROM ubuntu
   
   RUN apt-get update && apt install curl -y
   
   CMD ["echo" , "Hello from my custom image!" ]
   
   #4  : Added ubuntu in docker group 
   sudo usermod -aG docker $USER
   
   #5  : Build docker image
   docker build -t my-ubuntu:v1 .
   
   #6  : Ran docker image
   docker run my-ubuntu:v1
   
   #Result : Got result in terminal
   Hello from my custom image!
   
   ```

---

## Dockerfile Instructions

Create a new Dockerfile that uses **all** of these instructions:
- `FROM` — base image
- `RUN` — execute commands during build
- `COPY` — copy files from host to image
- `WORKDIR` — set working directory
- `EXPOSE` — document the port
- `CMD` — default command

   ```bash
   # Defining base image to use for our application
   FROM node:22-alpine

   # Defines working directory
   WORKDIR /app

   # Copied package.json and package-lock.json to working directory
   COPY package*.json ./

   # Installs project dependencies
   RUN npm install

   # Now copied the whole app
   COPY . .

   # Build the app
   RUN npm run build

   # Installs the serve package globally inside the container so the CMD can use it.
   RUN npm install -g serve

   # Exposes port 3000 for application to run on it
   EXPOSE 3000

   # Hosts our /dist folder and makes it accessible on port 3000
   CMD ["serve", "-s", "dist", "-l", "3000"]
   ```

---


## CMD vs ENTRYPOINT

1. Create an image with `CMD ["echo", "hello"]` — run it, then run it with a custom command. What happens?

   ```bash
   FROM alpine

   CMD ["echo" , "hello world"]
   ```

   - CMD
      - The default command that runs when a container starts. Can be completely overridden by passing a command at docker run.
      - Ex : docker run cmd-test echo bye     # Output: bye  ← CMD got replaced


2. Create an image with `ENTRYPOINT ["echo"]` — run it, then run it with additional arguments. What happens?

   ```bash
   FROM alpine

   ENTRYPOINT ["echo" , "Hello"]
   ```

   - ENTRYPOINT
      - The fixed command that always runs when a container starts. Arguments passed at docker run are appended to it, not replace it.
      - Ex : docker run ep-test World   # Output: Hello World     ← appended



3. Write in your notes: When would you use CMD vs ENTRYPOINT?
   - CMD : Default behaviour, only to use when user may want to override
   - ENTRYPOINT : When Container has one fixed purpose (always runs that command)



---

## Build a Simple Web App Image

1. Create a small static HTML file (`index.html`) with any content

2. Write a Dockerfile that:
   - Uses `nginx:alpine` as base
   - Copies your `index.html` to the Nginx web directory

3. Build and tag it `my-website:v1`

4. Run it with port mapping and access it in your browser


```bash
# Created an index.html file in /nginx-app directory
index.hmtl


# Created Dockerfile : 
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/

EXPOSE 80


# Build the image (with tag -> v1)
docker build -t my-website:v1

# Run the container
docker run --name nginx-web -d -p 3000:80 my-website:v1

# Opened browser and pasted the public url of ec2 with port 3000 (localhost:3000 for docker desktop) 
App index.html served successfully
```


---

## .dockerignore

1. Create a `.dockerignore` file in one of your project folders
2. Add entries for: `node_modules`, `.git`, `*.md`, `.env`
3. Build the image — verify that ignored files are not included

   ```bash
   # To list all files of existing container(hidden files too)
   docker exec a05 ls -a

   # Stop and remove existing container 
   docker stop a05 && docker rm a05

   # Created .dockerignore file 
   vim .dockerignore

   # Build the images again to exclude the files and folder we want 
   docker build -t todo-app:v3 .

   # Run the container
   docker run -d -p 8000:3000 todo-app:v3

   # Checked that .dockerignore is added succesfully and all the files and folders we added in it gets removed 
   docker exec 4fbdc ls -a
   ```
---

## Build Optimization

1. Build an image, then change one line and rebuild — notice how Docker uses **cache**
2. Reorder your Dockerfile so that frequently changing lines come **last**
3. Write in your notes: Why does layer order matter for build speed?

*Notes*

- Building images is slow — installing packages, compiling code takes time. 
- Cache lets Docker skip steps that haven't changed, making rebuilds fast.
- Before running each layer, Docker checks :
   - Has this instruction changed? Have the files involved changed?
      - If no → reuses the cached layer 
      - If yes → runs it fresh, and breaks cache for all layers below 

---
