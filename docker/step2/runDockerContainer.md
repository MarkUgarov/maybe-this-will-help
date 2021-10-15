# Run a Docker Container

## Check your images and Re-Tag (optional)

In your terminal (e.g. Command Prompt or IntelliJ-Terminal) execute

> docker images

It should show something like that:

TODO: Picture

If you don't see your image, go back to either [import the docker image](../step1/importDockerImage.md) or [create a new docker image](../step1/createOracleDockerImage.md)

If you are happy with name and tag, go to the next step. Otherwise you can rename it with  the `docker tag` -command (https://docs.docker.com/engine/reference/commandline/tag/).

> docker tag IMAGE_ID TARGET_IMAGE[:TAG]

## Prepare

### Prepare volume folder

Also create a directory which you want to mount for your database. It is actually not mandatory, but it prevents some errors that will appear if you database gets bigger.

### Prepare docker file

You can use docker run command or docker-compose. I will demonstrate with docker-compose, because I think it's much more readable, but you can also choose to evaluate the docker run command.

First create a file "docker-compose.yml" anywhere in your filesystem, I suggest to keep it somewhere near your project-directory (e.g in the parent folder, mine is named "FinHyb"). You can open it with any editor, but IntelliJ might be helpful.

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

Now replace
1. the <CONTAINER_NAME> with the name of the imported container
2. the <TAG> with the tag of the imported container (but leave the ":" between name and tag)
3. <MY_VOLUME_FOLDER> with the volume folder

## Run the container

Open your terminal (e.g. Command Prompt or IntelliJ-Terminal) and navigate to the folder that contains the "docker-compose.yml". Excute

> docker-compose up

It might take a while, but the image should be imported.

## Check if everything works

### Check log

Open Docker Desktop (in Windows: see in the far right of your taskbar, there should be the symbol somewhere, just left-click once).

You should see something that resembles the folder name you located your docker-compose.yml in (mine is showing "finhyb"). You can click on the little arrow left to it. Something should "unfold" and show "oracle19c". Click on it to see the console.

Hint: If you run into Problems, altneratively call

> docker logs -f --tail 100 oracle19c

to see the log. If there is no output, your container does not run.

### Change password

It might be necessary to change the password e.g. if there are shown in the log or you have issues with the following step "Setup FinHyb user". This should not happen, but my experience dissents.
Call

> docker exec -it oracle19c bash

or use docker desktop to open the terminal (CLI) of your container (the Symbol looks like ">.")

TODO: Picture

If the container runs, something is shown like

> \[oracle@1a9de9a580e1 ~\]$

With a blinking cursor. Congratulations, you are now inside the container which is Linux. You can not copy-paste via Ctrl+C / Ctrl+V between this console and your Windows while remaining here.  Execute

> ./setPassword.sh password

Hint: You can also change the password to something else, but then you have to adapt the one in the docker-compose.yml.

Stop the container and rerun with `docker-compose up`. See if the problems remain. If so, you're now on your own.

(Just kidding. Google it and if you don't find anything, ask around, don't be shy to talk to your teammates.)
