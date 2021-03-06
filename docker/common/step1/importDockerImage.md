# Getting a docker image from an archive

This page does not focus on pulling a docker container, because that is described many times. 
See e.g. [https://github.com/MaksymBilenko/docker-oracle-12c](https://github.com/MaksymBilenko/docker-oracle-12c), which you can pull via

> docker pull quay.io/maksymbilenko/oracle-12c 


**Hint**:
With docker-compose, images that are hosted via [docker hub](https://hub.docker.com/search?q=&type=image) don't need to be imported manually first!
They will get pulled automatically when executing `docker-compose up` as described in [Run an image as container](../step2/runDockerContainer.md).

---

## Step 1: Download / Copy / ...

There are many ways to get an archive. In most cases it will be be a `.tar`, like `oracle19c.tar`. 
Mabe a friend gave it to you on a LaserDisk after he [exported his docker container](../step3/exportindex.md)

Download / copy /... it anywhere. 

---

## Step 2: Import

Navigate to the folder where you downloaded the file via console (e.g. Command Prompt or IntelliJ-Terminal) you  execute the following

> docker load --input `<your archive name>`

e.g.
> docker load --input oracle19c.tar

This may take a while!

---

# Next steps
You might like to [run your docker image](../step2/runDockerContainer.md)