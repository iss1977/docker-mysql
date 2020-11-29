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


### Connecting to MySQL¶
Now that we know MySQL is up and running, let's use it! But, the question is... how? If we run another container on the same network, how do we find the container (remember each container has its own IP address)?
To figure it out, we're going to make use of the [nicolaka/netshoot](https://github.com/nicolaka/netshoot) container, which ships with a lot of tools that are useful for troubleshooting or debugging networking issues.
1. Start a new container using the nicolaka/netshoot image. Make sure to connect it to the same network.
```
docker run -it --network todo-app nicolaka/netshoot
```

2. Inside the container, we're going to use the `dig` command, which is a useful DNS tool. We're going to look up the IP address for the hostname `mysql`.
```
dig mysql
```
And you'll get an output like this...
```text
; <<>> DiG 9.14.1 <<>> mysql
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32162
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;mysql.             IN  A

;; ANSWER SECTION:
mysql.          600 IN  A   172.23.0.2

;; Query time: 0 msec
;; SERVER: 127.0.0.11#53(127.0.0.11)
;; WHEN: Tue Oct 01 23:47:24 UTC 2019
;; MSG SIZE  rcvd: 44
```
In the "ANSWER SECTION", you will see an A record for `mysql` that resolves to `172.23.0.2` (your IP address will most likely have a different value). While mysql isn't normally a valid hostname, Docker was able to resolve it to the IP address of the container that had that network alias (remember the `--network-alias` flag we used earlier?).
What this means is... our app only simply needs to connect to a host named `mysql` and it'll talk to the database! It doesn't get much simpler than that!

### Running our App with MySQL

The todo app supports the setting of a few environment variables to specify MySQL connection settings. They are:
- MYSQL_HOST - the hostname for the running MySQL server
- MYSQL_USER - the username to use for the connection
- MYSQL_PASSWORD - the password to use for the connection
- MYSQL_DB - the database to use once connected

With all of that explained, let's start our dev-ready container!

We'll specify each of the environment variables above, as well as connect the container to our app network.
```
docker run -dp 3000:3000 `
  -w /app -v "$(pwd):/app" `
 --network todo-app `
 -e MYSQL_HOST=mysql `
 -e MYSQL_USER=root `
 -e MYSQL_PASSWORD=secret `
 -e MYSQL_DB=todos `
 node:12-alpine `
  sh -c "yarn install && yarn run dev"
```
If we look at the logs for the container `docker logs <container-id>`, we should see a message indicating it's using the mysql database.
```
# Previous log messages omitted
$ nodemon src/index.js
[nodemon] 1.19.2
[nodemon] to restart at any time, enter `rs`
[nodemon] watching dir(s): *.*
[nodemon] starting `node src/index.js`
Connected to mysql db at host mysql
Listening on port 3000
```

Open the app in your browser and add a few items to your todo list.
Connect to the mysql database and prove that the items are being written to the database. Remember, the password is secret.

```
mysql> select * from todo_items;
+--------------------------------------+--------------------+-----------+
| id                                   | name               | completed |
+--------------------------------------+--------------------+-----------+
| c906ff08-60e6-44e6-8f49-ed56a0853e85 | Do amazing things! |         0 |
| 2912a79e-8486-4bc3-a4c5-460793a575ab | Be awesome!        |         0 |
+--------------------------------------+--------------------+-----------+
```





