# Run a Docker Container with Oracle DB

## Step 0: Check your images and Re-Tag (optional)

In your terminal (e.g. Command Prompt or IntelliJ-Terminal) execute

> docker images

It should show something like that:

TODO: Picture

If you don't see your image, go back to either [import the docker image](../step1/importDockerImage.md) or [create a new docker image](../step1/createOracleDockerImage.md)

If you are happy with name and tag, go to the next step. Otherwise you can rename it with  the `docker tag` -command (see [https://docs.docker.com/engine/reference/commandline/tag/](https://docs.docker.com/engine/reference/commandline/tag/)).

> docker tag `<IMAGE_ID TARGET_IMAGE:TAG>`
 

---

## Step 1: Prepare

### Prepare volume folder

Also create a directory which you want to mount for your database. It is actually not mandatory, but it prevents some errors that will appear if your database gets bigger.

### Prepare docker file

You can use docker run command or docker-compose. I will demonstrate with docker-compose, because I think it's much more readable, but you can also choose to evaluate the docker run command.

First create a file "docker-compose.yml" anywhere in your filesystem, I suggest to keep it somewhere near your project-directory (e.g in the parent folder). You can open it with any editor, but IntelliJ might be helpful.

Paste the follwing in it:

```yaml
version: '3.3'
  services:
    oracle19c:
      container_name: oracle19c
      image: <CONTAINER_NAME>:<TAG>
      restart: always
      volumes:
        - "<MY_VOLUME_FOLDER>:/opt/oracle/oradata"
      environment:
        - ORACLE_PDB=orcl
        - ORACLE_PWD=password
        - ORACLE_MEM=4000
        - ORACLE_SID=ORCLCDB
        - COMPOSE_CONVERT_WINDOWS_PATHS=1
        ports:
          - "1521:1521"
          - "5500:5500"
volumes:
  db_data: { }
```

Run the following line to see the details of your imported image.

> docker images

Now replace inside the yaml:
1. the <CONTAINER_NAME> with the name of the imported container
2. the <TAG> with the tag of the imported container (but leave the ":" between name and tag)
3. <MY_VOLUME_FOLDER> with the volume folder
4. optional: the `oracle19c` with any name you like (because you actually work with a newer version or whatever)

Hint: You can also change the password to something else, but you have to consider it later when we [configure the database](../../oracle/configureOracleDbInContainer.md).

---

## Step 2: Run the container

Open your terminal (e.g. Command Prompt or IntelliJ-Terminal) and navigate to the folder that contains the `docker-compose.yml`. Execute

> docker-compose up

It might take a while, but the image should be started.

---

## Step 3: Check if everything works

Open Docker Desktop (in Windows: see in the far right of your taskbar, there should be the symbol somewhere, just left-click once).

You should see something that resembles the folder name you located your docker-compose.yml in. You can click on the little arrow left to it. Something should "unfold" and show "oracle19c". Click on it to see the console.

Hint: If you run into Problems, alternatively call

> docker logs -f --tail 100 `<your container name>`

to see the log. If there is no output, your container does not run.

Mine is 

> docker logs -f --tail 100 oracle19c

---

# Next steps

You might want to [configure the database](../../oracle/configureOracleDbInContainer.md).