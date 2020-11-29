## Creating a Docker container with MySQL persistent data

We want to create a container with a server, and another one with mysql.
We also want to create **persistent data** for our MySQL container.

First, we create a Network, so our containers can talk to each other.

`docker network create todo-app-network`

Then run the following command in Powershell:
```
docker run -d `
    --network todo-app-network --network-alias mysql `
    -v todo-mysql-data:/var/lib/mysql `
    -e MYSQL_ROOT_PASSWORD=secret `
    -e MYSQL_DATABASE=todos `
    mysql:5.7
```
After running this command we will get an running mysql instance 

> Note, that docker will automatically create a docker volume called `todo-mysql-data` linked to the `/var/lib/mysql` folder of our mysql image.
> Run `docker volume ls` to see the volume `todo-mysql-data`

---

As next step, we want to **login in our MySQL instance**

Run `docker ps` to get our < mysql-container-id> : 

| CONTAINER ID | IMAGE | COMMAND | CREATED | STATUS | PORTS | NAMES
|---|---|---|---|---|---|---|
| **1e4f74226825** | mysql:5.7 | "docker-entrypoint.s…" | 5 minutes ago | Up 5 minutes | 3306/tcp, 33060/tcp | intelligent_meninsky |

Connect to MySQL:

```javascript
docker exec -it <mysql-container-id> mysql -p
```
*We set the password as “secret” when we started the container.* `<mysql-container-id>` *is* `1e4f74226825`

```sql
mysql> SHOW DATABASES;

+--------------------+  
| Database           |  
+--------------------+  
| information_schema |  
| mysql              |  
| performance_schema |  
| sys                |  
| todos              |  
+--------------------+  
5 rows in set (0.00 sec)  
```



