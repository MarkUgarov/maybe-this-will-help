# PostgreSQL database dump 

On some occasions, you don't want to export (or import) a docker image, but a database from a running container.

This might be the case if you want to exchange your DB with one of your colleagues, restore from a backup or update the version of your container.

Doing so with PostgreSQL is pretty easy since version 10.

**Don't have a container?**

Maybe you first want to look how to [create a PostgreSQL database container](creatingAPostrgreSQLDockerContainer.md).

## Export a database of a PostgreSQL docker container into a dump

Open your terminal (e.g. Command Prompt or IntelliJ-Terminal) and navigate to a folder where you want to put your dump into.

Execute

```bash
docker exec -i <container id> pg_dump  -U <DB_USERNAME> -F t <DB_NAME> > dump.sql.tar
```
(while - of course - replacing the `<DB_USERNAME>` and `<DB_NAME>`)

This should take a while. Afterwards, a new file `dump.sql.tar` should be created. You might want to send it to somebody or store it as a backup.

**Troubles?**

If the size is small (e.g. 1KB), something probably got wrong. 
In this case, execute 

```bash
docker exec -i <container id> pg_dump  -U <DB_USERNAME> -F t <DB_NAME> 
```
and look what happens.

## Import a dump into an existing PostgreSQL docker container

In this scenario, you already have a `dump.sql.tar` (maybe sent to you by somebody else).

Open your terminal (e.g. Command Prompt or IntelliJ-Terminal) and navigate to the folder where you placed your `dump.sql.tar`.

Execute

```bash
docker exec -i <container id> pg_restore -U <DB_USERNAME> -F t -d <DB_NAME> < dump.sql.tar
```

---

## If you want to know what is going on exactly.... 

**About the `>`and `<`**
* `>` redirects the output of the previous command into a file (including errors)
* `<` passes the content of a file to the previous command

You might want to check other ways to redirect on [https://ss64.com/nt/syntax-redirection.html](https://ss64.com/nt/syntax-redirection.html).


**About everything that's going on _before_ the `>`and `<`**

I gave you some examples how to export and import, but really understand what's going on, maybe read [this](https://www.postgresql.org/docs/10/app-pgdump.html) and [this](https://www.postgresql.org/docs/10/app-pgrestore.html).

---

## Next steps
Maybe you want to
- [connect your PostgreSQL database tor your IntelliJ](connectWithIntelliJ.md)
- see [more about docker and PostgreSQL](postgresql.md)
