# Committing an Exporting

This page describes how to export a docker image or container to an archive. 

## Option 1: Export your container directly

See also [https://docs.docker.com/engine/reference/commandline/export/](https://docs.docker.com/engine/reference/commandline/export/).

Just go to any directory where you want to save the archive and execute

> docker export --output="`<your archive name>`" `<your container name>`

For example if your container is named `oracle19c`, execute

> docker export --output="oracle19c.tar" oracle19c

This might take a while!
That should be it. You're done.

Maybe you want to upload that new archive somewhere so others can enjoy.

## Option 2: Export an image

### Step 1: Commit the container as image to your local private registry (optional)

Also see [https://docs.docker.com/engine/reference/commandline/commit/](https://docs.docker.com/engine/reference/commandline/commit/).

Example: You created your docker image with name `oracle/database` and tag `19.3.0-ee` as described in [Create docker image from scratch](../step1/createOracleDockerImage.md)
Execute

> docker tag oracle/database:19.3.0-ee localhost:5000/ora19
> docker push localhost:5000/ora19

Of course, you can also vary the name (localhost:5000) or the tag (ora19) if you want before executing.

This will create a new image which you can than export. Check with

> docker images

### Step 2: Re-Name / Re-Tag your image

If you are unhappy with existing names or tags, you can rename it with the "docker tag"-command (https://docs.docker.com/engine/reference/commandline/tag/).

> docker tag <IMAGE_ID> <TARGET_IMAGE>:<TAG>

### Step 3: Export / Save your image

An image can be exported (or "saved" as a .tar archive). See [https://docs.docker.com/engine/reference/commandline/save/](https://docs.docker.com/engine/reference/commandline/save/) or execute

>docker save -o oracle19c.tar <IMAGE_ID>

(Of course you may alter the name of the output archive, e.g. to "oracle21.tar".)

This might take a while!
