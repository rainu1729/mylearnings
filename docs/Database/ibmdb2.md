# IBM DB2 Local Setup Guide (Docker)

This guide walks you through setting up an IBM DB2 database locally in a Docker container for testing and development.

---

## Run IBM DB2 with Docker

IBM provides a developer edition image called `db2_community` that is free for development use.

### Step 1: Run the DB2 Container
Use the following docker run command to pull and start the container:

```bash
docker run -itd \
    --name db2-local \
    --privileged \
    -p 50000:50000 \
    -e LICENSE=accept \
    -e DB2INST1_PASSWORD=YourSecurePassword123 \
    -e DBNAME=testdb \
    icr.io/db2_community/db2
```

* **`--privileged`**: Required for DB2 to manage instance processes and mount resources internally.
* **`-p 50000:50000`**: Maps the default DB2 communication port to your host.
* **`LICENSE=accept`**: Accept the DB2 community terms.
* **`DB2INST1_PASSWORD`**: Sets the password for the default instance owner (`db2inst1`).
* **`DBNAME=testdb`**: Creates an initial database named `testdb`.

---

## Verify and Connect

### Step 1: Check Container Status
It may take 1-2 minutes for DB2 to initialize the database:
```bash
docker logs -f db2-local
```
Wait until you see messages indicating the instance has successfully started.

### Step 2: Access Command Line Interface (db2cmd)
You can execute DB2 command line tools directly inside the container:
```bash
docker exec -it db2-local bash -c "su - db2inst1"
```

Once inside the shell as user `db2inst1`, connect to the database:
```bash
db2 connect to testdb
db2 "create table users (id int, name varchar(50))"
db2 "insert into users values (1, 'Alice')"
db2 "select * from users"
```
