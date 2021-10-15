# Configure your Oracle DB inside a running container

Hint: This page was written using a `oracle19c` container. Your container might be named different. Adapt!

## Step 1: Change password

It might be necessary to change the password e.g. if there are shown in the log or you have issues with the following step. This should not happen, but my experience dissents.
Call

> docker exec -it oracle19c bash

or use docker desktop to open the terminal (CLI) of your container (the Symbol looks like ">.")

TODO: Picture

If the container runs, something is shown like

>[oracle@1a9de9a580e1 ~]$

With a blinking cursor. Congratulations, you are now inside the container which is Linux. You can not copy-paste via Ctrl+C / Ctrl+V between this console and your Windows while remaining here.  Execute

>./setPassword.sh password

Hint: You can also change the password to something else, but then you have to adapt the one in the `docker-compose.yml`.

Stop the container and rerun with `docker-compose up`. See if the problems remain. If so, you're now on your own.

(Just kidding. Google it and if you don't find anything, ask around, don't be shy to talk to your teammates.)

# Step 2 (Optional): Setup a user

When initialised, there should be already a user "SYS" you can work with. However, in most cases it is useful to set another user.

In this example, the user will be called "brad". Adapt if you want to.

First, open the console. As described above, call

> docker exec -it oracle19c bash

or use docker desktop to open the terminal (CLI) of your container (the Symbol looks like ">.")

Then execute
$ sqlplus

It will require credentials.
Username is "SYS as SYSDBA"
Password should be "password" (unless you changed it in the `docker-compose.yml` and Step 1).

If the credentials do not work, go back to "Change password" and change the password.
You can check if the database is working by executing

SQL> SELECT * FROM TAB;

Then execute the follwing (only the commands which are makred "SQL>" have to be executed, the lines in between should be the response if everything was executed correctly).

SQL> create user brad identified by brad;
User created.
SQL> grant unlimited tablespace to brad;
Grant succeeded.
SQL> grant resource,connect,dba to brad;
Grant succeeded.
SQL> grant create session to brad;
Grant succeeded.
SQL> grant create table to brad;
Grant succeeded.
SQL> grant create view to brad;
Grant succeeded.
SQL> grant create trigger to brad;
Grant succeeded.
SQL> grant create sequence to brad;
Grant succeeded.
exit;

Done.

## Hint 1: If it shows that the user is already existing, you probably imported an image that is already set up.

## Hint 2: There are sometimes errors that can be googeled. E.g. you have to run

SQL> alter session set "_ORACLE_SCRIPT"=true;

if "ORA-65096: invalid common user or role name" is shown.

# Connect with IntelliJ (optional)

Open the project of your choice (e.g clx-fbk-r5-web-portal) and Open the "Database" Tool-Window (which is normally shown on the far right on the screen, else go to "View" > "Tool Windows" and click "Database").
Inside (on the top) there is a small button for the "Data Source Properties". Click so the Properties open and add a new Oracle Data Source (via the small "+").

1. In the URL paste "jdbc:oracle:thin:@localhost:1521:ORCLCDB". Some of the fields should now fill automatically.
2. Choose "Connection Type": SID
3. Choose "Driver": Oracle
4. Set "Authentication": "User & Password". Type in "brad" and "password" (or whatever user and password you set before).

Hint: if this does not work, try with User "SYS as SYSDBA". 

Click "Test Connection"

