# Oracle Database with docker

I work with Windows, so if you work with some other OS, sometimes some small adaptation might be needed, but the commands of docker and docker-compose will be absolutely the same.

---

# Step 0: Download and install Docker Desktop

Go to docker.com, download and install. 

You will need admin rights.

If you don't have any clue what docker is, maybe look [here](../common/docker.md).

---

# Step 1: Get the docker image

## Option 1:

Pull a docker image from a repository.
This needs no extra page, because it is short and there are plenty examples out there in the internet.

E.g. see [https://github.com/MaksymBilenko/docker-oracle-12c](https://github.com/MaksymBilenko/docker-oracle-12c)
You can pull this example with

> docker pull quay.io/maksymbilenko/oracle-12c

## Option 2:
[Use a provided Docker Image from archive](../common/step1/importDockerImage.md)

## Option 3:
[Create a new Docker image from scratch](../common/step1/createOracleDockerImage.md)

---

# Step 2: Run the Docker Container
[Run an image as container](../common/step2/runDockerContainer.md)

---

# Step 3: Configure your database
[Configure your oracle database](configureOracleDbInContainer.md)

