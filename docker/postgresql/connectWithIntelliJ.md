## Connect with IntelliJ 

The following describes how to connect a docker container containing a PostgreSQL database with IntelliJ. If you don't have a container yet, check [how to create one](creatingAPostrgreSQLDockerContainer.md).

Open the project of your choice with IntellIJ and Open the "Database" Tool-Window (which is normally shown on the far right on the screen, else go to "View" > "Tool Windows" and click "Database").
Inside (on the top) there is a small button for the "Data Source Properties". Click so the Properties open and add a new Oracle Data Source (via the small "+").

## Step 1

In the URL paste "jdbc:postgresql://localhost:5432/<DB_NAME>" (replacing <DB_NAME> by whatever you wrote in docker-compose.yaml). Some of the fields should now fill automatically.

## Step 2

Choose "Driver": PostgreSQL (if not already done by IntelliJ)

## Step 3

Set "Authentication": "User & Password". Type in your <DB_USERNAME> and <DB_PASSWORD> (e.g. "brad" and "password" (or whatever user and password you set before when setting up [docker-compose.yml](creatingAPostrgreSQLDockerContainer.md#step-1-prepare-docker-compose-file)).

Click "Test Connection". If successful, you might want to click "ok" and analyze your database.

---

You might like to
- [export an existing PostgreSQL database and import it into your new container](exportAndImport.md) or
- see [more about docker and PostgreSQL](postgresql.md)
