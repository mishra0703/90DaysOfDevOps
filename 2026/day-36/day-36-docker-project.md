# Day 36 – Docker Project: Dockerize a Full Application

## Task
Our today's goal is to **take a real application and Dockerize it end-to-end**.

---

## Picking Our App

We cloned a mern full-stack ecommerce app and Dockerize it. To practice how to work on a developer's project. 

---

## Write the Dockerfile

1. Create a Dockerfile for your application
2. Use a **multi-stage build** if applicable
3. Use a **non-root user**
4. Keep the image **small** — use alpine or slim base images
5. Add a `.dockerignore` file

Build and test it locally.


### Multi-Staged Dockerfile 

```bash
FROM node:22 AS builder

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY frontend/package*.json ./frontend/

RUN npm install --prefix frontend

COPY backend ./backend

COPY frontend ./frontend

RUN npm run build --prefix frontend




FROM node:22-alpine AS runner

WORKDIR /app

RUN addgroup -S nodejs && adduser -S nodeuser -G nodejs

COPY package*.json ./

RUN npm install --omit=dev

COPY --from=builder /app/backend ./backend

COPY --from=builder /app/frontend/dist ./frontend/dist

RUN chown -R nodeuser:nodejs /app

USER nodeuser

ENV NODE_ENV=production

EXPOSE 5000

CMD ["node" , "backend/server.js"]
```


### Size of this Docker Image : 

```bash
IMAGE               ID             DISK USAGE   CONTENT SIZE  
node:22             103199348179       1.64GB          425MB
node:22-alpine      9385cd9f3001        230MB         57.8MB
ecommercea-app:v9   654fb593facc        300MB         73.1MB
```



---

## Add Docker Compose

Write a `docker-compose.yml` that includes:
1. Your **app** service (built from Dockerfile)
2. A **database** service (Postgres, MySQL, MongoDB — whatever your app needs)
3. **Volumes** for database persistence
4. A **custom network**
5. **Environment variables** for configuration (use `.env` file)
6. **Healthchecks** on the database

```bash
name: ecommerce-app

services:

  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: mern-ecommerce-app
    restart: unless-stopped
    ports:
      - "5000:5000"
    environment:
      NODE_ENV: development
      PORT: 5000
      MONGO_URI: mongodb://mongo:27017/mern-ecommerce
      UPSTASH_REDIS_URL: redis://redis:6379
      ACCESS_TOKEN_SECRET: ${ACCESS_TOKEN_SECRET}
      REFRESH_TOKEN_SECRET: ${REFRESH_TOKEN_SECRET}
      CLOUDINARY_CLOUD_NAME: ${CLOUDINARY_CLOUD_NAME}
      CLOUDINARY_API_KEY: ${CLOUDINARY_API_KEY}
      CLOUDINARY_API_SECRET: ${CLOUDINARY_API_SECRET}
      STRIPE_SECRET_KEY: ${STRIPE_SECRET_KEY}
      CLIENT_URL: ${CLIENT_URL}
    depends_on:
      mongo:
         condition: service_healthy
      redis:
         condition: service_healthy
    networks:
      - app-network

  mongo:
    image: mongo:7
    container_name: app-database
    restart: unless-stopped
    environment:
      MONGO_INITDB_DATABASE: mern-ecommerce
    volumes:
      - mongo-data:/data/db
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "mongosh", "--quiet", "--eval", "db.adminCommand('ping').ok"]
      interval: 10s       
      timeout: 5s         
      retries: 5          
      start_period: 30s

  redis:
    image: redis:7-alpine
    container_name: app-redis
    restart: unless-stopped
    volumes:
      - redis-data:/data
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s       
      timeout: 5s         
      retries: 5          
      start_period: 30s

volumes:
  mongo-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```



---

## Ship It

1. Tag your app image
   - `docker tag ecommerce-app:latest mishra0703/ecommerce-app:latest`

2. Push it to Docker Hub
   - `docker push mishra0703/ecommerce-app:latest`

3. Share the Docker Hub link
   - `https://hub.docker.com/r/mishra0703/ecommerce-app`

