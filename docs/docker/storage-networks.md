# Storage and networks

## Volumes

```bash
docker volume create todo-db
docker run -dp 127.0.0.1:3000:3000 --mount type=volume,src=todo-db,target=/etc/todos getting-started
```

## Bind mounts

Mount the current folder in `/src`:

=== "Linux"

    ```bash
    docker run -it --mount type=bind,src="$(pwd)",target=/src ubuntu bash
    ```

=== "PowerShell"

    ```powershell
    docker run -it --mount "type=bind,src=$($pwd),target=/src" ubuntu bash
    ```

## Networks

Create a network and attach a MySQL container to it:

```bash
docker network create todo-app
```

```powershell title="PowerShell"
docker run -d `
    --network todo-app --network-alias mysql `
    -v todo-mysql-data:/var/lib/mysql `
    -e MYSQL_ROOT_PASSWORD=secret `
    -e MYSQL_DATABASE=todos `
    mysql:8.0
```

Connect to it:

```bash
docker exec -it <mysql-container-id> mysql -u root -p
```
