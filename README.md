# Oracle XE In Docker

## Usage

### Docker

**Start docker daemon:** ``$ sudo dockerd``

**Start Oracle:** ``$ docker-compose up``

**Stop docker:** Just ``CTRL + C`` it, it will stop gracefully

### Database

**Connect** using your SQL tool of choice to:
* server: ``localhost``
* port: ``1521`` (depends on port mapping in docker)
* service: ``XE``
* username = ``sys as sysdba`` (not you will want to create additional users under ``XEPDB1``)

Note for non-sysdba connections, connect to server ``XEPDB1``

Or use browser to access Oracle Enterprise Manager at https://localhost:5500/em
* **TODO** this isn't working, port is listening but not data back

Or **connect using SQLPlus**
1. On host:

	```
	$ docker exec -ti <container> bash

	```

1. Then inside database:

	```
	$ sqlplus sys/<password>@//localhost:1521/XE as sysdba
	```

**Create user**

As sys as sysdba:

```sql
-- make sure to use the PDB container
ALTER SESSION SET container = XEPDB1;

CREATE USER MYUSER IDENTIFIED BY mypassword;
ALTER USER MYUSER quota unlimited ON USERS;
GRANT CREATE SESSION TO MYUSER;
GRANT CREATE TABLE TO MYUSER;
GRANT CREATE TABLE, CREATE VIEW, CREATE PROCEDURE, CREATE SEQUENCE TO MYUSER;
```


## First-Time Installation

1. Install requirements on host:
	* docker
	* docker-compose

1. The following steps summarise instructions from:
	* https://github.com/oracle/docker-images/tree/main/OracleDatabase/SingleInstance
	* https://blogs.oracle.com/oraclemagazine/post/deliver-oracle-database-18c-express-edition-in-containers

1. Clone the git repo and run the script ``/buildDockerImage.sh -v 18.4.0 -x`` script to build docker images for Oracle XE

1. Update ``docker-compose.yml`` so the image and tag match the built image

1. On the host, the ``oradata`` directory must be writeable by the container (this is where data is persisted)

	```
	$ sudo chown 54321:54321 oradata
	```

1. Start the container and the password will be auto-generated on first start (or change it using the instructions on [Github](https://github.com/oracle/docker-images/tree/main/OracleDatabase/SingleInstance))


## Issues And Future Work

* Fix https://localhost:5500/em
* Container is memory hungry and will uses all of the hosts free memory. Workaround by limiting WSL2 memory:

	1. In command prompt: ``$ wsl --shutdown``
	1. Edit ``%USERPROFILE%\.wslconfig`` to be:

		```
		# Global WSL 2 config
		# See https://docs.microsoft.com/en-us/windows/wsl/wsl-config

		[wsl2]
		memory=6GB
		```

	1. Start WSL. Note shutdown from step 1 takes [about 8 seconds](https://docs.microsoft.com/en-us/windows/wsl/wsl-config#the-8-second-rule). The command ``$ wsl --list --running`` will confirm it has stopped.

