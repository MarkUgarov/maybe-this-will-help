# Create a new Docker image from scratch

## Download from oracle.com

Download the LINUX Database from from http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html.

Important: Don't use the Windows version, we need LINUX because it will be executed inside the Docker-Container which is Linux.

E.g. Download the Oracle Database 19c binary LINUX.X64_193000_db_home.zip from http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html

## Clone from github.com/oracle

1. Clone https://github.com/oracle/docker-images.
2. Put the zip of the database (e.g. LINUX.X64_193000_db_home.zip) in the SingleInstance/dockerfiles-Directory of the matching version (e.g. in OracleDatabase/SingleInstance/dockerfiles/19.3.0). Do not unzip it.
3. If https://github.com/oracle/docker-images/pull/1576 is not yet merged, edit OracleDatabase/SingleInstance/dockerfiles/19.3.0/dbca.rsp.tmpl, and change totalMemory=2048 to totalMemory=4000 or whatever value you want.
5. In Docker Desktop, update the allocated memory to a value more than the value above.

## Enable Windows Subsystem 2 for Linux (obligatory for Windows)

We do need to enable Windows Subsystem 2 for Linux. Open a power shell as admin user. Run

> dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

That's it. Well done.

## Build the image

Open a terminal (or use the power shell you just opened) and go to the directory where the git clone happened. Go further into OracleDatabase/SingleInstance/dockerfiles and execute

>./buildContainerImage.sh -v 19.3.0 -e

This will build the image, but not the container.

You might want to check if the image exist by calling

> docker images