# Create a new Docker image from scratch

## Info (i)

The following page was composed by some basic knowledge about docker and reading the following links:
- [https://github.com/steveswinsburg/oracle19c-docker](https://github.com/steveswinsburg/oracle19c-docker)
- [https://www.martinberger.com/2020/09/windows-10-wsl-2-docker-and-oracle-a-perfect-partnership/](https://www.martinberger.com/2020/09/windows-10-wsl-2-docker-and-oracle-a-perfect-partnership/)

## Step 1: Download from oracle.com

Download the LINUX Database from from `http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html`.

**Important**: Do _not_ use the Windows version, we need LINUX because it will be executed inside the Docker-Container which is Linux.

E.g. Download the Oracle Database 19c binary `LINUX.X64_193000_db_home.zip`.

## Step 2: Clone from github.com/oracle

1. Clone `https://github.com/oracle/docker-images`.
2. Put the zip of the database you just downloaded (e.g. `LINUX.X64_193000_db_home.zip`) in the `SingleInstance/dockerfiles-Directory` of the matching version (e.g. in `OracleDatabase/SingleInstance/dockerfiles/19.3.0`). **Do not unzip it.**
3. Optional*: edit `OracleDatabase/SingleInstance/dockerfiles/19.3.0/dbca.rsp.tmpl`, and change total memory to at least 2048 (totalMemory=2048 to totalMemory=4000 or whatever value you want).
4. In Docker Desktop, update the allocated memory to a value at least 2048 (or the value you just set).

* Point 3 is optional in most cases, but obligatory if `https://github.com/oracle/docker-images/pull/1576` is not yet merged


## Step 2.8214: Enable Windows Subsystem 2 for Linux (obligatory for Windows)

We do need to enable Windows Subsystem 2 for Linux. Open a power shell as admin user. Run

> dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

That's it. Well done.

## Step 3: Build the image

To make the rest of the page more readable, I will continue with the Oracle 19c example. If you have any other version, please adapt the commands.

Open a terminal (or use the power shell you just opened) and go to the directory where the git clone happened. 

Go further into `OracleDatabase/SingleInstance/dockerfiles` and execute

>./buildContainerImage.sh -v 19.3.0 -e

This will build the image, but not the container.

You might want to check if the image exist by calling

> docker images

# Next steps
You might like to [run your docker image](../step2/runDockerContainer.md)
