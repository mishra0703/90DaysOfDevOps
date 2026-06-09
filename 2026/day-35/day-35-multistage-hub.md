# Day 35 – Multi-Stage Builds & Docker Hub

## Task
Our today's goal is to **build optimized images and share them with the world**.

Multi-stage builds are how real teams ship small, secure images. Docker Hub is how you distribute them. Both are interview favourites.

---

## The Problem with Large Images

1. Write a simple Go, Java, or Node.js app (even a "Hello World" is fine)
   - We used our vaultify nextjs full-stack app 

2. Create a Dockerfile that builds and runs it in a **single stage**

### Single Stage Dockerfile

   ```bash
   FROM node:22

   WORKDIR /app

   COPY package*.json ./

   RUN npm install

   COPY . .

   # I didn't knew about ARG and ENV so that's where I took help of claude
   ARG NEXTAUTH_SECRET
   ARG NEXTAUTH_URL
   ARG NEXT_PUBLIC_ENCRYPTION_KEY
   ARG ENCRYPTION_SECRET

   ENV NEXTAUTH_SECRET=$NEXTAUTH_SECRET
   ENV NEXTAUTH_URL=$NEXTAUTH_URL
   ENV NEXT_PUBLIC_ENCRYPTION_KEY=$NEXT_PUBLIC_ENCRYPTION_KEY
   ENV ENCRYPTION_SECRET=$ENCRYPTION_SECRET
   ENV NODE_OPTIONS="--max-old-space-size=512"


   RUN npm run build

   EXPOSE 3000

   CMD ["npm" , "start"]
   ```

### docker-compose.yml 

```bash
services:

  app:
    build:
      context: .           # Defines the build context directory
      args:                # Defines build-time variables that are passed directly to your Dockerfile 
        NEXTAUTH_SECRET: ${NEXTAUTH_SECRET}
        NEXTAUTH_URL: ${NEXTAUTH_URL}
        NEXT_PUBLIC_ENCRYPTION_KEY: ${NEXT_PUBLIC_ENCRYPTION_KEY}
        ENCRYPTION_SECRET: ${ENCRYPTION_SECRET}
    ports:
      - "3000:3000"
    environment:           # define and pass environment variables directly into a running container
      MONGODB_URI: mongodb://mongo:27017/vaultify
      NEXTAUTH_SECRET: ${NEXTAUTH_SECRET}
      NEXTAUTH_URL: ${NEXTAUTH_URL}
      NEXT_PUBLIC_ENCRYPTION_KEY: ${NEXT_PUBLIC_ENCRYPTION_KEY}
      ENCRYPTION_SECRET: ${ENCRYPTION_SECRET}
    depends_on:            # We added depends_on so it wait for mongo db to run 
      - mongo

  mongo:
    image: mongo:7
    ports:
      - "27017:27017"
    volumes:               # We attached volume to it so our data must stay safe
      - mongo_data:/data/db

volumes:
  mongo_data:
```


3. Build the image and check its **size**
   ```
   IMAGE                                        ID             DISK USAGE   CONTENT SIZE
   node:22                                     103199348179       1.64GB          425MB
   mongo:7                                     4b5bf3c2bb75       1.19GB          296MB
   vaultify-app:latest                         d21a62de2e85       3.28GB          827MB
   ```


*Notes* : 

- A stage in Docker refers to a distinct phase in a multi-stage Dockerfile, introduced using the FROM instruction.
- Normally, a single FROM = one stage. Multi-stage builds let you have multiple FROM blocks in one Dockerfile, each being a separate stage.
- Problem is everything used to build the app stays in the final image — Node.js toolchain, dev dependencies, build cache, npm itself
- Results in very large images (3GB+ for a Next.js app)
- **Important**  : Always `COPY package*.json` first, then `RUN npm ci`, then `COPY . .` . This way npm install only re-runs when dependencies actually change, not on every code change
- Environment variables in Next.js builds
   - `Next.js` bakes some env vars at build time, not just runtime
   - So it must be passed as `ARG` → `ENV` before the build step, not just at runtime
- Why ARG → ENV in Dockerfile
   - During docker build, there is no .env file available. So you have to explicitly pass the variable into the build process:
      ```bash
      ARG NEXT_PUBLIC_ENCRYPTION_KEY          # accept it as a build argument
      ENV NEXT_PUBLIC_ENCRYPTION_KEY=$ARG     # make it available to npm run build
      RUN npm run build                       # now Next.js can see it and bake it in
      ```
   - Without this, npm run build runs with the variable empty → broken bundle.
- Also our app is getting an Error
   - `SIGKILL` during npm run build because of `out of memory` and this is because ec2 has only 1GB Ram and very less space available 


---

## Multi-Stage Build

1. Rewrite the Dockerfile using **multi-stage build**:
   - Stage 1: Build the app (install dependencies, compile)
   - Stage 2: Copy only the built artifact into a minimal base image (`alpine`, `distroless`, or `scratch`)
2. Build the image and check its size again
3. Compare the two sizes

Write in your notes: Why is the multi-stage image so much smaller?


### Modified Dockerfile (Multi-stage Dockerfile)

```bash

# Stage 1: Building the app (install dependencies, compile)
## Its only job is to produce the .next/ folder. That's it. Everything it does is in service of that one output:

# -> Sets up the full Node.js environment
# -> Installs every dependency including dev deps
# -> Receives all environment variables
# -> Runs npm run build to compile the app
# -> Dies — never ships to production

# Think of it as a construction site. Cranes, scaffolding, cement mixers, workers — all present during construction, none of it is part of the finished building.



FROM node:22 AS builder

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

ARG NEXTAUTH_SECRET
ARG NEXTAUTH_URL
ARG NEXT_PUBLIC_ENCRYPTION_KEY
ARG ENCRYPTION_SECRET

ENV NEXTAUTH_SECRET=$NEXTAUTH_SECRET
ENV NEXTAUTH_URL=$NEXTAUTH_URL
ENV NEXT_PUBLIC_ENCRYPTION_KEY=$NEXT_PUBLIC_ENCRYPTION_KEY
ENV ENCRYPTION_SECRET=$ENCRYPTION_SECRET
ENV NODE_OPTIONS="--max-old-space-size=512"

RUN npm run build





# Stage 2: Copy only the built artifact into a minimal base image (`alpine`, `distroless`, or `scratch`)

## Its only job is to serve the already-built app:

# -> Starts completely fresh with a minimal Alpine image
# -> Copies only the built output (.next/) from stage 1
# -> Installs only production dependencies
# -> Runs npm start to serve the app

# No build tools, no source code, no dev dependencies, no compiler — just the finished product running.


FROM node:22-alpine AS runner

WORKDIR /app

COPY --from=builder /app/package*.json ./

RUN npm install

COPY --from=builder /app/.next ./.next

COPY --from=builder /app/public ./public

EXPOSE 3000

CMD ["npm" , "start"]
```



### New Image Size after building it from Multi-Stage Dockerfile

```
IMAGE                                        ID             DISK USAGE   CONTENT SIZE

vaultify-app:latest                         d21a62de2e85        491MB          827MB
```


### Why is the multi-stage image so much smaller?

- Alpine vs full Node image
   - node:22 is Debian-based and ships with compilers, package managers, system tools (In short it is a heavy base image). Whereas node:22-alpine is stripped to the bare minimum (Means lighter base image). That one swap saves ~340MB before you've done anything else.

- Dev dependencies are left behind in stage 1 typescript, eslint, webpack, turbopack, @types/* — all the packages only needed to build the app never enter stage 2.
   
- Source code and build tools never enter stage 2. Your src/, app/, components/ folders, the npm cache, the TypeScript compiler — none of it is copied. Stage 2 only receives the finished compiled output via COPY --from=builder.


**Summary :** Multi-stage is smaller because stage 1 is the factory (heavy, temporary, thrown away) and stage 2 is only the finished product — Alpine base + prod dependencies + built output. The build tools, dev dependencies, source code, and npm cache never make it into the final image.


---

## Push to Docker Hub
1. Create a free account on [Docker Hub](https://hub.docker.com) (if you don't have one)
   - Created

2. Log in from your terminal
   - Run `docker login` , it will start web based login 

3. Tag your image properly: `yourusername/image-name:tag`
   - `docker tag vaultify-app:latest mishra0703/vaultify:latest`
   - Docker Hub needs it renamed in the format username/imagename:tag
   - This doesn't create a new image — it just adds a new name/alias pointing to the same image. Your original vaultify-app:latest still exists. As we can see both `vaultify-app:latest` and `mishra0703/vaultify:latest` will have same image id.

4. Push it to Docker Hub
   - `docker push mishra0703/vaultify:latest`
   - Docker uploads each layer individually. Layers already existing on Docker Hub get skipped — only new ones upload.
   - We can verify the upload on docker hub website : `https://hub.docker.com/repositories/mishra0703`

5. Pull it on a different machine (or after removing locally) to verify
   - `docker pull mishra0703/vaultify:latest` : Will work only after `docker rmi` to verify



---

## Docker Hub Repository

1. Go to Docker Hub and check your pushed image
   - It's there

2. Add a **description** to the repository
   - Next.js full-stack app with MongoDB & NextAuth, containerized with multi-stage Docker build.
   - Added

3. Explore the **tags** tab — understand how versioning works
   - A tag is just a label pointing to a specific version of your image.
   - Same image, different labels. Like renamed shortcuts to the same file.

      ```
      mishra0703/vaultify:latest    ← "latest" is the tag
      mishra0703/vaultify:v1.0      ← "v1.0" is the tag
      mishra0703/vaultify:stable    ← "stable" is the tag
      ```

4. Pull a specific tag vs `latest` — what happens?
   - Pulls whatever latest points to `docker pull mishra0703/vaultify`
   - Pulls exactly v1.0 — guaranteed same image forever `docker pull mishra0703/vaultify:v1.0`



---

## Image Best Practices

1. Use a **minimal base image** (alpine vs ubuntu — compare sizes)

   ```bash
   # Before — full debian node image

   FROM node:22                  # node:22 (debian) ~400MB (full OS + tools)


   # After — alpine (bare minimum linux)

   FROM node:22-alpine           # node:22-alpine ~60MB (minimal linux) 
   ```


2. **Don't run as root** — add a non-root USER in your Dockerfile

   ```bash
   FROM node:22-alpine

   # Create a system group + user

   RUN addgroup --system --gid 1001 nodejs \       # System Group

       && adduser --system --uid 1001 nextjs       # System User


   COPY --from=builder /app/.next ./.next

   RUN npm ci --omit=dev


   # switch to non-root before starting

   USER nextjs


   EXPOSE 3000

   CMD ["npm", "start"]
   ```

- root — full container access if exploited
- nextjs user — limited permissions only
   

***Why this matters :** Because by-default every process in a container runs as root (uid 0). If your app gets exploited, the attacker has root inside the container. With USER nextjs, they get a system user with no home directory, no shell, no sudo — far less damage possible.*



3. Combine `RUN` commands to **reduce layers**

   ```bash

   # Before — 3 separate layers

   RUN apt-get update

   RUN apt-get install -y curl

   RUN rm -rf /var/lib/apt/lists/*                 # delete in layer 3 ≠ removed from layer 2



   # After — 1 layer, cleanup in same step

   RUN apt-get update \

       && apt-get install -y curl \

       && rm -rf /var/lib/apt/lists/*              # install + delete in same step

   ```

***The trap :** If you install a package in one RUN and delete it in the next RUN, the file still exists in the earlier layer — Docker stores every layer permanently. The image size doesn't shrink. Combining into one RUN means the deletion happens before the layer is committed, so the file never gets stored.*


4. Use **specific tags** for base images (not `latest`)

   ```bash
   # Floating — changes without warning

   FROM node:latest                    # latest — today Node 22, tomorrow Node 23
                                  

   # Major version only — minor can still change

   FROM node:22                        # node:22 — minor patches can still change
 

   # Exact — same image every single build

   FROM node:22.3.0-alpine             # node:22.3.0 — locked forever, reproducible
   ```


***Real scenario :** Your pipeline uses node:latest. One morning Docker Hub updates latest to a new major version. Your build picks it up silently. Something breaks in production. You spend hours debugging — the code didn't change, the base image did. Pinning to node:22.3.0-alpine means this can never happen.*

