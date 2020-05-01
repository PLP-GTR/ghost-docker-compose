# ghost-docker-compose
Ghost blog using docker compose

Following the guide _"Hosting a Ghost blog with NGINX and Docker"_ from Alistair Chapman:

* Part 1: https://blog.agchapman.com/ghost-blog-with-nginx-and-docker-1/
* Part 2: https://blog.agchapman.com/ghost-blog-with-nginx-and-docker-2/

## Problems I came across

### SSL companion

#### Docker socket missing

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

#### CA authorization invalid

After ghost was running (as docker said), I couldn't access it via registered domain. So I've restarted the nginx & ssl-companion to see this error:
```
ssl-companion    | 2020-04-30 15:32:35,816:ERROR:simp_le:1417: CA marked some of the authorizations as invalid, which likely means it could not access http://example.com/.well-known/acme-challenge/X. Did you set correct path in -d example.com:path or --default_root? Are all your domains accessible from the internet? Please check your domains' DNS entries, your host's network/firewall setup and your webserver config. If a domain's DNS entry has both A and AAAA fields set up, some CAs such as Let's Encrypt will perform the challenge validation over IPv6. If your DNS provider does not answer correctly to CAA records request, Let's Encrypt won't issue a certificate for your domain (see https://letsencrypt.org/docs/caa/). Failing authorizations: https://acme-v02.api.letsencrypt.org/acme/authz-v3/4276342995
```
Well... obviously, since I couldn't ping the new domain and neither could some online service. I've added the subdomain some days ago, so it should be accessible. But now I've discovered, that the second-level-domain had a different name server attached. I thought this would not have any effect, but sure it did. After adding the 4th level domain to this service, the DNS is working correctly. The domain is currently `staging.3rdlevel.2ndlevel.tld` shaped.

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

#### Aborted DB connections

After ghost configured itself and created the admin user, the following warnings occurred:
```
[...]
blog_1     | ghost   INFO  ==> Creating admin user...
blog_1     | ghost   INFO  The admin user was created
mariadb_1  | 2020-04-30 15:12:07 31 [Warning] Aborted connection 31 to db: 'bitnami_ghost' user: 'bn_ghost' host: '172.19.0.3' (Got an error reading communication packets)
mariadb_1  | 2020-04-30 15:12:07 28 [Warning] Aborted connection 28 to db: 'bitnami_ghost' user: 'bn_ghost' host: '172.19.0.3' (Got an error reading communication packets)
mariadb_1  | 2020-04-30 15:12:07 27 [Warning] Aborted connection 27 to db: 'bitnami_ghost' user: 'bn_ghost' host: '172.19.0.3' (Got an error reading communication packets)
mariadb_1  | 2020-04-30 15:12:07 20 [Warning] Aborted connection 20 to db: 'bitnami_ghost' user: 'bn_ghost' host: '172.19.0.3' (Got an error reading communication packets)
mariadb_1  | 2020-04-30 15:12:07 26 [Warning] Aborted connection 26 to db: 'bitnami_ghost' user: 'bn_ghost' host: '172.19.0.3' (Got an error reading communication packets)
mariadb_1  | 2020-04-30 15:12:07 25 [Warning] Aborted connection 25 to db: 'bitnami_ghost' user: 'bn_ghost' host: '172.19.0.3' (Got an error reading communication packets)
mariadb_1  | 2020-04-30 15:12:07 24 [Warning] Aborted connection 24 to db: 'bitnami_ghost' user: 'bn_ghost' host: '172.19.0.3' (Got an error reading communication packets)
mariadb_1  | 2020-04-30 15:12:07 23 [Warning] Aborted connection 23 to db: 'bitnami_ghost' user: 'bn_ghost' host: '172.19.0.3' (Got an error reading communication packets)
mariadb_1  | 2020-04-30 15:12:07 30 [Warning] Aborted connection 30 to db: 'bitnami_ghost' user: 'bn_ghost' host: '172.19.0.3' (Got an error reading communication packets)
mariadb_1  | 2020-04-30 15:12:07 29 [Warning] Aborted connection 29 to db: 'bitnami_ghost' user: 'bn_ghost' host: '172.19.0.3' (Got an error reading communication packets)
```
_Not fixed yet._ **UPDATE:** The errors did not show up again but I think I force-recreated the containers in the meantime. I guess some not completed setup process might have caused this.

#### Directory permission denied

Some time after the initialization and the abroted connection warnings, this one came up:\
`blog_1     | Error executing 'postInstallation': EACCES: permission denied, mkdir '/bitnami/ghost'`

So I've granted too many permissions to the persistence volume mount:\
`/var/docker/ghost$ sudo chmod -R 777 ghost-persistence/`

But this just lead to another error with instant container crash:
```
blog_1     | Error executing 'postInstallation': ELOOP: too many symbolic links encountered, lstat '/opt/bitnami/ghost/logs/ghost.log'
ghost_blog_1 exited with code 1
```

This was indeed easily fixed by shutting the containers down and bringing them up again - instead of just restarting the previous, crashed one:
```
/var/docker/ghost$ docker-compose down
Removing ghost_blog_1    ... done
Removing ghost_mariadb_1 ... done
Removing network ghost_default
/var/docker/ghost$ docker-compose up
```

So now, I've got to this point:
```
blog_1     | - Starting Ghost: 172-20-0-3
blog_1     | ✔ Starting Ghost: 172-20-0-3
blog_1     |
blog_1     | ------------------------------------------------------------------------------
blog_1     |
blog_1     | Your admin interface is located at:
blog_1     |
blog_1     |     http://172.20.0.3/ghost/
```

### Nginx

#### No live upstreams

After all the above problems where fixed, trying to access the domain lead to:
```
nginx-proxy      | nginx.1    | 2020/05/01 18:47:55 [error] 175#175: *44 no live upstreams while connecting to upstream, client: 188.xxx.xxx.xxx, server: staging.3rdlevel.2ndlevel.tld, request: "GET / HTTP/2.0", upstream: "http://staging.3rdlevel.2ndlevel.tld/", host: "staging.3rdlevel.2ndlevel.tld"
```

Checking the nginx config in the container tells me, that the exposed port of ghost, in my case `2368` seems not to be used.\
Config location: `/etc/nginx/conf.d/default.conf`.\
As of the guide, which states that _By default, the config will point at the only port that the blog service exposes: 2368_ it should work. I have to investigate why a) nginx is not automatically using the port or b) where I have to tell him.
