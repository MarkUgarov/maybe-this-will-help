# Creating a PostgreSQL docker container ("postgres")

There are [several versions](https://hub.docker.com/_/postgres) available on docker hub for Postgres. 

---

## Step 1: Prepare docker-compose file

Create a file 'docker-compose.yaml'. It may be laying in a project folder, but theoretically it can be anywhere.

Copy the following into this file:

```yaml
version: '3'
services:
    postgresql-db:
        container_name: <CONTAINER_NAME> 
        image: postgres:12.3
        command: [ "postgres", "-c", "log_statement=all" ] 
        restart: always
        volumes:
           # maybe you want to replace 'postgre_volume' in the following line with a path on your local system 
           # (e.g. C:\Users\YOU\postgresql\YOUR_PROJECT\data)
            - postgre_volume:/var/lib/postgresql/data  
        environment:
            - POSTGRES_PASSWORD=<DB_PASSWORD>
            - POSTGRES_USER=<DB_USERNAME>
            - POSTGRES_DB=<DB_NAME>
            - PGDATA=/var/lib/postgresql/data/pgdata
        ports:
            - 5432:5432
volumes:
    postgre_volume: {}
```

Now replace 
1. <CONTAINER_NAME> with the name you want your container to appear with, e.g. 'postgresql'
2. <DB_PASSWORD> with any password you like, e.g. "password"
3. <DB_USERNAME> with the username you like to access your database with, e.g. "brad"
4. <DB_NAME> with some database name

---

## Step 2: Run the container

You don't actually have to [import the image](../common/step1/importDockerImage.md) since the image is available on docker hub. 

Open your terminal (e.g. Command Prompt or IntelliJ-Terminal) and navigate to the folder that contains the `docker-compose.yml` from [Step 1](#step-1-create-a-docker-compose-file).

Execute

> docker-compose up

It might take a while, but the image should be imported and started.

---

## Step 3: Check if everything works

Open Docker Desktop (in Windows: see in the far right of your taskbar, there should be the symbol somewhere, just left-click once).

You should see something that resembles the folder name you located your docker-compose.yml in. You can click on the little arrow left to it. Something should "unfold" and show whatever you replaced the `<CONTAINER_NAME>` by (e.g. 'postgresql').  
Click on it to see the console.

Hint: If you run into Problems, alternatively call

> docker logs -f --tail 100 `<CONTAINER_NAME>`

while (of course) replacing `<CONTAINER_NAME>`, e.g. 

> docker logs -f --tail 100 postgresql

to see the log. If there is no output, your container does not run.

---

# Next steps
You might like to 
- [export an existing PostgreSQL database and import it into your new container](exportAndImport.md) or
- [connect to the database of your container with IntelliJ](connectWithIntelliJ.md)
- see [more about docker and PostgreSQL](postgresql.md)

