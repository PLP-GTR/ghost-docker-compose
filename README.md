# ghost-docker-compose
Ghost blog using docker compose

Following the guide _"Hosting a Ghost blog with NGINX and Docker"_ from Alistair Chapman:

* Part 1: https://blog.agchapman.com/ghost-blog-with-nginx-and-docker-1/
* Part 2: https://blog.agchapman.com/ghost-blog-with-nginx-and-docker-2/

## Problems I came across

### SSL companion

Setting up the ssl companion like this:

```
  ssl-companion:
    image: jrcs/letsencrypt-nginx-proxy-companion
    container_name: ssl-companion
    volumes:
      - /apps/web/ssl:/etc/nginx/certs:rw # same path as above, now RW
    volumes_from:
      - proxy
```

Left me with the error:\
`ssl-companion | Error: you need to share your Docker host socket with a volume at /var/run/docker.sock`.\
As suggested [here](https://github.com/nginx-proxy/docker-letsencrypt-nginx-proxy-companion/issues/87#issuecomment-235324412) I've added the volume:\
`/var/run/docker.sock:/var/run/docker.sock:ro`

### Maria DB

I came across this error: `db_1    | mkdir: cannot create directory '/bitnami/mariadb': Permission denied`.\
Using this volume:

```
    volumes:
      - /var/docker/ghost/mariadb-persistence:/bitnami
```

So I've ran `/var/docker/ghost$ sudo chmod -R 777 mariadb-persistence/`.\
There might be a better way, which I might figure out some other time.

### Ghost

#### DB connection error

This error occured: `blog_1  | Error executing 'postInstallation': Failed to connect to mariadb:3306 after 36 tries`.\
Manually adding the hostname `mariadb` to the container fixed the issue:
```
services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    environment:
      - HOSTNAME=mariadb
```

#### DB access failure

Now, the connection was successful but ghost couldn't login to maria db server:
```
blog_1     | mysql-c INFO  Found MySQL server listening at mariadb:3306
mariadb_1  | 2020-04-30 14:46:30 9 [Warning] Access denied for user 'bn_ghost'@'172.19.0.3' (using password: NO)
blog_1     | mysql-c ERROR [canConnect] Connection with 'bn_ghost' user is unsuccessful
```

An environment information with the DB password fixed this:
```
      - GHOST_DATABASE_PASSWORD=
```
