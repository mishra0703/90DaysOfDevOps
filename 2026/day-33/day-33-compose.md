# Day 33 – Docker Compose: Multi-Container Basics

## Task
Our today's goal is to **run multi-container applications with a single command**.

Yesterday we manually created networks and volumes and ran containers one by one. Docker Compose does all of that in one YAML file.

---

## Install & Verify

1. Check if Docker Compose is available on your machine
    - `docker compose` : Will show the list of commands/flags available to use with docker compose which shows that it is available on our machine

2. Verify the version
    - `docker compose version` : Shows the version of docker compose

---

## Our First Compose File

1. Create a folder `compose-basics`
    - `mkdir compose-basics && cd compose-basics`

2. Write a `docker-compose.yml` that runs a single **Nginx** container with port mapping
    - The headache of first making a dockerfile then creating an image after that running that image as a container is gone with docker-compose
    - `docker compose up -d` = `docker run -d --name my-nginx -p 8080:80 nginx` ———> just written in a file instead of a long command.
    ```bash
    services:
    web:
      image: nginx
      container_name: my-nginx
      ports:
        - "8080:80"
    ```

3. Started it with `docker compose up` or `docker compose up -d` to run it in backgound    

4. Accessed it in browser at `100.53.191.37:8080` (Public IP of ec2 instance with port no. 8080 as we chose it to serve our nginx app which runs on port 80)

5. Stopped it with `docker compose down`

---

## Two-Container Setup

Write a `docker-compose.yml` that runs:
- A **WordPress** container
- A **MySQL** container

They should:
- Be on the same network (Compose does this automatically)
- MySQL should have a named volume for data persistence
- WordPress should connect to MySQL using the service name

Start it, access WordPress in your browser, and set it up.

**Verify:** Stop and restart with `docker compose down` and `docker compose up` — is your WordPress data still there?

```bash
services:
  db:
    image: mysql:8.0
    container_name: wp-mysql
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wp123
    volumes:
      - db-data:/var/lib/mysql          # Docker Volume : db-data is attached to mysql container
    command: --default-authentication-plugin=mysql_native_password

  wordpress:
    image: wordpress:latest
    container_name: wp-app
    ports:
      - "8080:80"                       # We can access our app at port 8080
    environment:
      WORDPRESS_DB_HOST: db             # Service name of mysql (Line no. 70)
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wp123
    depends_on:
      - db                              # wp-app will only run if service db or we can say mysql runs successfully

volumes:
  db-data:
```

- We tested wp-app by doing `docker compose up` and then `docker compose down` and then doing again `docker compose up` to see if our created wordpress account is still there or not 
- Only because we mapped docker volume to mysql's database our created account is still there even after docker compose down
- Only because we ran both containers in docker-compose they could connect to each other 


---

## Compose Commands

1. Start services in **detached mode**
    - `docker compose up -d` ———> flag `-d` ensures that the service should run in background

2. View running services
    - `docker ps` or `docker compose ps`

3. View **logs** of all services
    - `docker compose logs`
    - `docker compose logs -f` to follow logs live
    
4. View logs of a **specific** service
    - `docker compose logs <serve_name>` 
    - Here , for sql run `docker compose logs db` as db is the service name of mysql container (Line no. 70) , and for wordpress run `docker compose logs wordpress` as wordpress is the service name of wordpress container (Line no. 82)

5. **Stop** services without removing
    - `docker compose stop` or to stop any specific service only `docker compose stop <service_name>`

6. **Remove** everything (containers, networks)
    - `docker compose down` will stop and remove everything at once  

7. **Rebuild** images if you make a change
    - `docker compose up -d --build` to run the containers and rebuild the images that we changed 
    - `docker compose build` rebuild without running containers
    - Without `--build`, docker compose up reuses the existing image even if you changed your code. So, always add `--build` after making changes to your Dockerfile or app code.
    

---

## Environment Variables

1. Add environment variables directly in your `docker-compose.yml`
2. Create a `.env` file and reference variables from it in your compose file
3. Verify the variables are being picked up


### docker-compose.yml file

```bash
services:
  sql-app:
    image: mysql:8.0
    container_name: my-sql-db
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}       # This will takes the value from .env file
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    command: --default-authentication-plugin=mysql_native_password
    volumes:
      - sql-data:/var/lib/mysql

  app:
    image: alpine
    container_name: my-alpine-app
    environment:
      DB_HOST: sql-app
      DB_NAME: ${MYSQL_DATABASE}        # This will takes the value from sql-app (Line no. 152)
      DB_USER: ${MYSQL_USER}
      DB_PASS: ${MYSQL_PASSWORD}
    command: sleep infinity
    depends_on:
      - sql-app

volumes:
  sql-data:
```


### .env file

```bash
MYSQL_ROOT_PASSWORD=root123
MYSQL_DATABASE=testdb
MYSQL_USER=devuser
MYSQL_PASSWORD=dev123
APP_PORT=8080
```

### Verifying that env file's values got used by app

```bash
docker exec my-alpine-app env | grep DB         # This will show all the values 

# Output : 
DB_NAME=testdb
DB_USER=devuser
DB_PASS=dev123
DB_HOST=sql-app
```


