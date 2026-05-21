# Day 32 – Docker Volumes & Networking

## Task
Our today's goal is to **solve two real problems: data persistence and container communication**.

Containers are ephemeral — they lose data when removed. And by default, containers can't easily talk to each other. Today we will fix both.

---

## The Problem

1. Run a Postgres or MySQL container
2. Create some data inside it (a table, a few rows — anything)
3. Stop and remove the container
4. Run a new one — is your data still there?


```bash
## Before making and adding the volume to sql container

# Run a simple mysql container
docker run --name mysql -e MYSQL_ROOT_PASSWORD=1234 -d mysql

# Bash into that container (Open bash terminal in that container)
docker exec -it be8 bash

### In bash 

# Log in to mysql using root user and root user's password
mysql -u root -p
---Enter Password---

# See all databases
show databases;

Output : 
+--------------------+
| Database           |  # The below databases are default databases of mysql container 
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+

# Now create a new one and see if it has been created or not
create database imp_data;

show databases;

Output : 
+--------------------+
| Database           |
+--------------------+
| information_schema |
| imp_data           |      # New database has been created
| mysql              |
| performance_schema |
| sys                |
+--------------------+

# Now stop the container and then delete/remove it 
docker stop be8 && docker rm be8

# Now when we create a new container again and go to mysql terminal and check all the databases we will see that our database that we have been created is been removed there's no sign of it 

```

---

## Named Volumes

1. Create a named volume
    - `docker volume create backup_volume`

2. Run the same database container, but this time **attach the volume** to it
    -  `docker run --name mysql -v backup_volume:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=1234 -d mysql`

3. Add some data, stop and remove the container
    - As we did above 
        - Create database in mysql container
        - verify if it is created
        - stop and remove the container

4. Run a brand new container with the **same volume**
    -  `docker run --name mysql-new -v backup_volume:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=4321 -d mysql`

5. Is the data still there?
    - Yes our created database is still there as it is , no changes happened to it 



## The Solution

```bash
# So , Docker volume management is critical because it decouples data from the ephemeral lifecycle of containers. Without volumes, any data created inside a container is lost forever when the container is deleted or updated. As we have seen 



### Now we will see what happens when we attach a volume to the container 

# First Create a docker volume
docker volume create mysql-volume

# You can see the volumes list using 
docker volume ls


# Then do the same process as we did , with just a small change in docker run command
# Add -v flag to map volume with the container's data storage folder 
docker run --name mysql -v mysql-volume:/var/lib/mysql  -e MYSQL_ROOT_PASSWORD=1234 -d mysql


# Now when you do the same process of creating a database in sql container , then stop and delete the container and run the new container again and you'll see your data is still there
 
+--------------------+
| Database           |
+--------------------+
| imp_data           |      # This will stay even after deleting the container 
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

---

## Bind Mounts
1. Create a folder on your host machine with an `index.html` file
    - `mkdir index.html-file` in working directory (Where Dockerfile is present) 

2. Run an Nginx container and **bind mount** your folder to the Nginx web directory
    - Made a Dockerfile for the same 
    ```bash
    FROM nginx:alpine

    COPY  index.html-file/index.html /usr/share/nginx/html/

    EXPOSE 80
    ```
    - `docker run -d -v /home/ubuntu/nginx-app/index.html-file/:/usr/share/nginx/html/ -p 3000:80 nginx-app:v4`

3. Access the page in your browser
    - Opened `18.204.55.66:3000` in browser , the app is running fine

4. Edit the `index.html` on your host — refresh the browser
    - Editted the index.html file on host machine , and saw the changes on browser after refreshing the page.


### Difference between a Named Volume and a Bind Mount?

#### Named Volume
- Docker manages the storage location on your host. You just give it a name.
- Stored at /var/lib/docker/volumes/mydata/ — you don't control the path.

*Use named volumes for databases in production.*


#### Bind Mount
- You specify an exact folder on your host machine to map into the container.
- Whatever is in /home/ubuntu/data on your machine is directly visible inside the container.

*Use bind mounts when you want live code changes reflected inside the container during development.*


---

## Docker Networking Basics

1. List all Docker networks on your machine
    - `docker network ls` : Lists all the networks present on our machine

2. Inspect the default `bridge` network
    - `docker network inspect <bridge_id>` : Inspects bridge network

#### Notes

- When we install Docker, it automatically creates a virtual network called bridge on our machine. Every container we run gets connected to it by default.
- Docker acts as a mini router at 172.17.0.1 (Docker Bridge Gateway IP) and assigns each container an IP like 172.17.0.2, 172.17.0.3 etc.
- This is because containers can talk to the internet and to our host without any setup. Out of the box networking.

    ```bash
               Your Machine
    ┌─────────────────────────────────┐
    │  Container A    Container B     │
    │  172.17.0.2     172.17.0.3      │
    │        \           /            │
    │         bridge (172.17.0.1)     │
    │              |                  │
    │           Host Machine          │
    │              |                  │
    │           Internet              │
    └─────────────────────────────────┘
    ```

3. Run two containers on the default bridge — can they ping each other by **name**?
    - `docker exec container-a ping container-b` 
    - ping: bad address 'container-b'
    - Because Default bridge has no DNS — containers don't know each other's names, they only knows each other by IPs.


4. Run two containers on the default bridge — can they ping each other by **IP**?
    - Find their IPs
    - `docker exec container-a ping 172.17.0.3` {We only have to write target container's IP}
    - 64 bytes from 172.17.0.3: seq=0 ttl=64   # Works!  


---

## Custom Networks

1. Create a custom bridge network called `my-app-net`
    - `docker network create local-net`
    - `docker network ls` : Lists all network

2. Run two containers on `my-app-net`
    - `docker run -d --name container-a --network local-net alpine sleep infinity`
    - `docker run -d --name container-b --network local-net alpine sleep infinity`


3. Can they ping each other by **name** now?
    - `docker exec container-a ping container-b`  :   Yes they now ping by name too

4. Write in your notes: Why does custom networking allow name-based communication but the default bridge doesn't?
    - Docker's default bridge was built early on, before DNS was added to Docker. It's kept as-is for backward compatibility. No DNS resolver runs on it — containers only know IPs.
    - Custom bridge networks have built-in DNS — containers resolve each other by name automatically.
    - When we create a network , and when running a container just connect all the containers with same network. Every container joined to that network registers its name with it's DNS server. So when container-a pings container-b by name, DNS resolves it to the right IP automatically.
        ```bash
        Default Bridge              Custom Bridge
        ┌─────────────┐             ┌─────────────────────┐
        │ container-a │             │ container-a          │
        │ container-b │             │ container-b          │
        │             │             │ DNS: 127.0.0.11  ✅  │
        │ No DNS ❌   │             │ resolves names→IPs   │
        └─────────────┘             └─────────────────────┘
        ping by name = ❌           ping by name = ✅
        ping by IP   = ✅           ping by IP   = ✅
        ```



---

## Put It Together

1. Create a custom network
    - `docker network create app-net`

2. Run a **database container** (MySQL/Postgres) on that network with a volume for data
    - `docker run -d --name sql-db --network app-net -e MYSQL_ROOT_PASSWORD=root123 -e MYSQL_DATABASE=testdb -v app-vol:/var/lib/mysql mysql:8.0 --default-authentication-plugin=mysql_native_password --require-secure-transport=OFF`
    - This command is to run a `mysql container` using `mysql:8.0` as base image
    - It has network --> app-net 
    - It has volume --> app-vol
    - some plugin and secure authentication flags (Couldn't able to connect the app without it)
    
3. Run an **app container** (use any image) on the same network
    - `docker run -d --name alpine-app --network app-net alpine sleep infinity`
    - It also has the same network as the one database container has
    - `sleep infinity` keep the app running without doing anything.
    
*More Step*
-  Run it inside the alpine-app shell.
-  `docker exec -it alpine-app sh`
- `apk add mysql-client` install mysql-client inside shell 
- `mysql -h sql-db -u root -proot123 testdb --ssl=0` : This will take us inside testdb that is attached to mysql too 
- Create some data you want 
- Ex : 
    ```bash 
    CREATE TABLE tasks (id INT, title VARCHAR(50));
    INSERT INTO tasks VALUES (1, 'Learn Docker');
    SELECT * FROM tasks;
    EXIT;
    ```


4. Verify the app container can reach the database by container name
    - `docker exec alpine-app nc -zv sql-db 3306` : Verify by running this command 
        - `sql-db (172.19.0.2:3306) open`  ---> Means app and database is connected  
        -  `docker exec alpine-app ping sql-db` ---> To check weather they can be ping by name or not (Possible only if they are on the same network and connected via it's DNS)
    - In 3rd point , see *More Steps* there you'll see that we went into alpine-app's shell but accessed sql-db container's database and created data succesfully. That is also a proof 



## Screenshot Proof

![Alt Text](Screenshot%20Proof.png)