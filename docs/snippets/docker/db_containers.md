# Database containers

Each container keeps its data in a named volume.

!!! warning
    The passwords are examples for a lab. Change them before using these containers for anything else.

=== "MySQL"

    ```bash
    docker run -d -p 3306:3306 --name mysql-db -e MYSQL_ROOT_PASSWORD=123456 -v mysqldata:/var/lib/mysql mysql
    ```

=== "Oracle"

    User: `sys`

    ```bash
    docker run --name oracle -p 1521:1521 -e ORACLE_PWD=123456 -v oracledb:/opt/oracle/oradata container-registry.oracle.com/database/free:latest
    ```

=== "PostgreSQL"

    ```bash
    docker run -p 5432:5432 -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=123456 -e POSTGRES_DB=postgres -v pgdata:/var/lib/postgresql/data -d postgres
    ```

=== "SQL Server"

    ```bash
    docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=Password_123#" -p 1433:1433 -v sqlvolume:/var/opt/mssql --name sql_server_container -d mcr.microsoft.com/mssql/server
    ```

=== "MongoDB"

    ```bash
    docker run --name mongodb -d -p 27017:27017 -v mongodbdata:/data/db mongodb/mongodb-community-server:latest
    ```
