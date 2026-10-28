# Database containers

Each container keeps its data in a named volume.

!!! warning
    The passwords are examples for a lab. Change them before using these containers for anything else.

!!! tip
    The images use a fixed version tag, so everyone in the class gets the same version. Change the tag to upgrade.

=== "MySQL"

    ```bash
    docker run -d -p 3306:3306 --name mysql-db -e MYSQL_ROOT_PASSWORD=123456 -v mysqldata:/var/lib/mysql mysql:8.4
    ```

=== "Oracle"

    The password is used for `SYS`, `SYSTEM` and `PDBADMIN`. The pluggable database is `FREEPDB1` (see [Oracle](../oracle.md)).

    ```bash
    docker run -d --name oracle -p 1521:1521 -e ORACLE_PWD=123456 -v oracledb:/opt/oracle/oradata container-registry.oracle.com/database/free:latest
    ```

=== "PostgreSQL"

    ```bash
    docker run -d --name postgres -p 5432:5432 -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=123456 -e POSTGRES_DB=postgres -v pgdata:/var/lib/postgresql postgres:18
    ```

=== "SQL Server"

    User: `sa`. The password must be at least 8 characters with upper case, lower case, numbers and symbols.

    ```bash
    docker run -d --name sql_server_container -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=Password_123#" -p 1433:1433 -v sqlvolume:/var/opt/mssql mcr.microsoft.com/mssql/server:2022-latest
    ```

=== "MongoDB"

    ```bash
    docker run --name mongodb -d -p 27017:27017 -v mongodbdata:/data/db mongodb/mongodb-community-server:latest
    ```
