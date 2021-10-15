# Connect with IntelliJ (optional)

Open the project of your choice with IntellIJ and Open the "Database" Tool-Window (which is normally shown on the far right on the screen, else go to "View" > "Tool Windows" and click "Database").
Inside (on the top) there is a small button for the "Data Source Properties". Click so the Properties open and add a new Oracle Data Source (via the small "+").

1. In the URL paste "jdbc:oracle:thin:@localhost:1521:ORCLCDB". Some of the fields should now fill automatically.
2. Choose "Connection Type": SID
3. Choose "Driver": Oracle
4. Set "Authentication": "User & Password". Type in "brad" and "password" (or whatever user and password you set before when setting up [docker-compose.yml](../common/step2/runDockerContainer.md#prepare-docker-file) and [database](configureOracleDbInContainer.md#step-1-change-password)).

Hint: if this does not work, try with User "SYS as SYSDBA". The password should also be "password". 

Click "Test Connection"

---

# Done?
Maybe you want to check [all steps of running an oracle database via docker-compose](../../oracle/oracle.md)
or you would like to [export your container](../common/step3/exportDocker.md#option-1-export-your-container-directly)