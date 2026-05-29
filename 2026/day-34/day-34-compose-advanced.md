# Day 34 – Docker Compose: Real-World Multi-Container Apps

## Task
Our today's goal is to **build more complex, production-like setups with Docker Compose**.

Yesterday was basics. Today we will handle real scenarios — app + database + cache, healthchecks, restart policies, and service dependencies.

---

## Build our Own App Stack

Create a `docker-compose.yml` for a 3-service stack:
- A **web app** (use Python Flask, Node.js, or any language you know)
- A **database** (Postgres or MySQL)
- A **cache** (Redis)

Write a simple Dockerfile for the web app. The app doesn't need to be complex — even a "Hello World" that connects to the database is enough.


## *Notes app using express , mysql and redis*


### App Structure 


```
notes-app/
├── docker-compose.yml
├── .env
└── app/
    ├── Dockerfile
    ├── package.json
    └── index.js
```


## app/package.json

```bash
{
  "name": "notes-app",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "node index.js"
    },
  "dependencies": {
    "express": "^4.18.2",
    "mysql2": "^3.6.0",
    "redis": "^4.6.7",
    "cors": "^2.8.5"
    },
}
```


## app/index.js

```bash
import express from 'express'
import mysql from 'mysql2/promise'
import { createClient } from 'redis'
import cors from 'cors'

const app = express()
app.use(express.json())
app.use(cors())


# MySQL connection
const connectWithRetry = async () => {
  let retries = 10
  while (retries) {
    try {
      const db = await mysql.createConnection({
        host: process.env.DB_HOST,
        user: process.env.DB_USER,
        password: process.env.DB_PASS,
        database: process.env.DB_NAME
      })
      console.log('-----------MySQL connected-----------')
      return db
    } catch (err) {
      retries--
      console.log(`MySQL not ready, retrying... (${retries} left)`)
      await new Promise(res => setTimeout(res, 3000))
    }
  }
  throw new Error('Could not connect to MySQL')
}

const db = await connectWithRetry()


# Redis connection

const redis = createClient({ url: `redis://${process.env.REDIS_HOST}:6379` })
await redis.connect()


# Create table if not exists
await db.execute(`
  CREATE TABLE IF NOT EXISTS notes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  )
`)


# GET all notes — check Redis cache first
app.get('/notes', async (req, res) => {
  const cached = await redis.get('notes')
  if (cached) {
    console.log('Serving from Redis cache')
    return res.json({ source: 'cache', notes: JSON.parse(cached) })
  }
  const [rows] = await db.execute('SELECT * FROM notes ORDER BY created_at DESC')
  await redis.setEx('notes', 30, JSON.stringify(rows)) # cache for 30 seconds
  console.log('Serving from MySQL')
  res.json({ source: 'database', notes: rows })
})


# POST new note — clear cache so fresh data loads
app.post('/notes', async (req, res) => {
  const { content } = req.body
  await db.execute('INSERT INTO notes (content) VALUES (?)', [content])
  await redis.del('notes')   # invalidate cache
  res.json({ message: 'Note saved!' })
})


# DELETE a note — clear cache
app.delete('/notes/:id', async (req, res) => {
  await db.execute('DELETE FROM notes WHERE id = ?', [req.params.id])
  await redis.del('notes')  
  res.json({ message: 'Note deleted!' })
})


app.listen(3000, () => console.log('App running on port 3000'))
```


### app/Dockerfile


```bash
FROM node:18-alpine

WORKDIR /app


COPY package.json .

RUN npm install


COPY . .


EXPOSE 3000


CMD ["node", "index.js"]
```



### .env file

```bash
DB_HOST=mydb
DB_USER=devuser
DB_PASS=dev123
DB_NAME=notesdb
DB_ROOT_PASSWORD=root123
REDIS_HOST=redis
```



### docker-compose.yml

```bash
services:
  app:
    build: ./app
    container_name: notes-app
    ports:
      - "3000:3000"
    environment:
      DB_HOST: ${DB_HOST}
      DB_USER: ${DB_USER}
      DB_PASS: ${DB_PASS}
      DB_NAME: ${DB_NAME}
      REDIS_HOST: ${REDIS_HOST}
    
  mydb:
    image: mysql:8.0
    container_name: mydb
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
      MYSQL_DATABASE: ${DB_NAME}
      MYSQL_USER: ${DB_USER}
      MYSQL_PASSWORD: ${DB_PASS}
    command: --default-authentication-plugin=mysql_native_password
    volumes:
      - db-data:/var/lib/mysql

  redis:
    image: redis:alpine
    container_name: redis

volumes:
  db-data:
```


---

## depends_on & Healthchecks
1. Add `depends_on` to your compose file so the app starts **after** the database
2. Add a **healthcheck** on the database service
3. Use `depends_on` with `condition: service_healthy` so the app waits for the database to be truly ready, not just started

**Test:** Bring everything down and up — does the app wait for the DB?


### New docker-compose.yml


```bash
services:
  app:
    build: ./app
    container_name: notes-app
    ports:
      - "3000:3000"
    environment:
        ...same as before 
    depends_on:             # Added depends_on to app so that it will wait everytime for database to become ready
      mydb :
        condition: service_healthy

  mydb:
    image: mysql:8.0
    container_name: mydb
    environment:
        ...same as before
    healthcheck:                        # HealthCheck Added to mysql (database)
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-p${DB_ROOT_PASSWORD}"]     # Healthcheck Command
      interval: 10s             # Frequency of the check
      timeout: 5s               # Maximum time to wait for a response
      retries: 5                # Number of failures before marking as unhealthy
      start_period: 30s         # Delay before the first check (gives MySQL time to boot)
    command: --default-authentication-plugin=mysql_native_password
    volumes:
      - db-data:/var/lib/mysql
```


---

## Restart Policies

1. Add `restart: always` to your database service
2. Manually kill the database container — does it come back?
3. Try `restart: on-failure` — how is it different?
4. Write in your notes: When would you use each restart policy?

```bash
services:
  app:
    build: ./app
    container_name: notes-app
    ...same as it was
    depends_on:
      mydb:
        condition: service_healthy
      redis:
        condition: service_started
    restart: on-failure         # Added restart: on-failure so that it will get auto. restart on crash or failure of the app

  mydb:
    image: mysql:8.0
    container_name: mydb
    ...same as it was
    volumes:
      - db-data:/var/lib/mysql
    restart: always             # Added restart: always for mysql db so that even on kill it will auto. gets restart 
    
  redis:
    image: redis:alpine
    container_name: redis
    restart: always             # Added restart: always for reddis
```

### **Note** : 

- When we added restart policies to our services , the `restart: always` policy does not seem to work. when I kill the container (simulating app crash using docker kill) but docker-compose does not restart my container, even though the `Exit Code is 137`. I observe the same behaviour when I use `restart: on-failure` policy too. 

- So the thing is when we use docker kill, this is the expected behavior as Docker does not restart the container: "If you manually stop a container, its restart policy is ignored until the Docker daemon restarts or the container is manually restarted". This is an attempt to prevent a restart loop by docker itself

- But if I restart the daemon...

- The container that was set with restart policy, starts again which is what documentation say, so docker kill is not the way you should test the restart policy as it's assumed that you have deliberately stopped the container and Docker wants to have a way to prevent restarting loops, if you kill it, you really want to kill it.

- Summary : Restart policies protect against unexpected crashes, not intentional stops | `docker kill` = intentional | Process dying inside = crash | Only crashes trigger restart policy. 



### When to use which restart policy 

- Production DB → `restart: always`
- Dev environment → `restart: unless-stopped`
- Scripts/jobs → `restart: on-failure`
- One-time tasks → `restart: no`


---


## Custom Dockerfiles in Compose

1. Instead of using a pre-built image for your app, use `build:` in your compose file to build from a Dockerfile
    - We've already added `build:` in our compose file to build from the Dockerfile present at app/Dockerfile for notes-app


2. Make a code change in your app
    ```bash
    # Added this in index.js
    
    app.get('/health', (req, res) => {
      res.json({ status: 'ok', message: 'Notes app is running!' })
    })
    ```    


3. Rebuild and restart with one command
    - Then run `docker compose up -d --build` or `docker compose up -d --build app` to only build app not mysql not redis


---

## Named Networks & Volumes

1. Define **explicit networks** in your compose file instead of relying on the default
    - Without explicit network (what we had)
        - Compose auto-creates one default network — all services join it. No control over structure.


2. Define **named volumes** for database data
    - We've already added named volume and mapped it with our mysql container so that our data will stay safe

3. Add **labels** to your services for better organization


```bash
services:
  app:
    build: ./app
    container_name: notes-app
    ...same as it was
    networks:
      - frontend
      - backend
    labels:
      app.name: "notes-app"
      app.tier: "frontend"
      app.version: "1.0"
    restart: on-failure

  mydb:
    image: mysql:8.0
    container_name: mydb
    ...same as it was
    networks:
      - backend
    labels:
      app.name: "notes-db"
      app.tier: "database"
      app.version: "1.0"
    restart: always

  redis:
    image: redis:alpine
    container_name: redis
    networks:
      - backend
    labels:
      app.name: "notes-cache"
      app.tier: "cache"
      app.version: "1.0"
    restart: always

networks:
  frontend:
    name: notes-frontend
  backend:
    name: notes-backend
```

```
frontend network        backend network
┌─────────────┐        ┌──────────────────────────┐
│   app  <───────────────>  app + mydb + redis    │
└─────────────┘        └──────────────────────────┘
```

- `mydb` and `redis` are only on backend — not directly accessible from outside, only through `app`.
-  By running docker network ls , we can see the networks we added in docker-compose.yml 

```bash
# By running docker network inspect notes-backend we saw

 "Containers": {
            "113b6c229c2c4b967eacc65c07aeb0c906274064aa9ffa39242dd329fff8c212": {
                "Name": "redis",
                "EndpointID": "d283381445b5f7921ef7b26f62733b09b6052ff9ac7ca42cfdc2f19e10227de4",
                "MacAddress": "d2:5c:5d:f3:40:29",
                "IPv4Address": "172.20.0.3/16",
                "IPv6Address": ""
            },
            "9f0f09964f4a3371ef6174d86a2642a334e00168d86c343c7e55d1ff0222f56d": {
                "Name": "notes-app",
                "EndpointID": "1886360a1c51622d87c6ec4ff2094d1c605647deface2c83dfb4ec65b2a71c4e",
                "MacAddress": "a2:4f:a9:05:40:7f",
                "IPv4Address": "172.20.0.4/16",
                "IPv6Address": ""
            },
            "b25585c6cf3d31951091811ca8f7bfdc6195ab066ada2a76f7f3ddd2f861e455": {
                "Name": "mydb",
                "EndpointID": "2737a6be07c2a35495df83508b54e6721bea7b5a0c070fb933ccbfe5cc2f5300",
                "MacAddress": "ce:c1:e2:74:11:dc",
                "IPv4Address": "172.20.0.2/16",
                "IPv6Address": ""
            }
        },

# Means our all three (app , mydb and redis) containers are connected to each other via this single network
```


- Explicit networks matter : `mydb` is only on backend — even if someone accesses our `app`, they can't directly reach the database network. It's a security boundary. 

- Labels are metadata — they don't affect runtime but help in filtering, monitoring, and organizing containers in large projects.

---

## Scaling (Bonus)

1. Try scaling your web app to 3 replicas using `docker compose up --scale`
2. What happens? What breaks?
3. Write in your notes: Why doesn't simple scaling work with port mapping?


- `docker compose up -d --scale app=3` : Scaling to 3 replicas
- Error we got : `Error: container name "notes-app" is already in use`
- Fix 
    ```bash
    app:
      build: ./app
      # remove container_name ← delete this line
      ports:
        - "3000"      # ← no host port, Docker assigns random ones
    ```

- After doing `docker compose up -d --scale app=3` do `docker ps`  ---> we'll see 3 app containers with different random ports
    ```bash
    NAME              IMAGE                STATUS                   PORTS
    notes-app-app-1   notes-app-app      Up 7 minutes             0.0.0.0:32771->3000/tcp, [::]:32771->3000/tcp
    notes-app-app-2   notes-app-app      Up 7 minutes             0.0.0.0:32772->3000/tcp, [::]:32772->3000/tcp
    notes-app-app-3   notes-app-app      Up 7 minutes             0.0.0.0:32773->3000/tcp, [::]:32773->3000/tcp
    mydb              mysql:8.0          Up 7 minutes (healthy)   3306/tcp, 33060/tcp
    redis             redis:alpine       Up 7 minutes             6379/tcp
    ---

- We can see there are three containers with different names and random ports


- Simple scaling doesn't work with port mapping

    - When we map `host_port:container_port` , we're binding a `specific port` on our machine to one container. A machine has only one port 3000 — you can't split it between 3 containers.  

    - This is why real scaling needs a `Load Balancer` in front — it sits on port 3000 and distributes traffic across all replicas behind it. Docker Compose alone isn't built for production scaling — that's what `Kubernetes` and `Docker Swarm` are for.

- Importance of `Load Balancer` : 
    ```
    Without Load Balancer (broken)     With Load Balancer (real scaling)

    User → port 3000 → ??? 	           User → port 3000 → Load Balancer
                                                              ├── app replica 1
                                                              ├── app replica 2
                                                              └── app replica 3    
    ```


**Summary** : Port mapping ties one host port to one container — scaling needs many containers sharing one entry point, which requires a load balancer. Docker Compose scaling is for testing only; production scaling is Kubernetes territory.

*Note* : Also we've to open those random ports in our EC2 Security Group inbound rules , to hit any request.


---