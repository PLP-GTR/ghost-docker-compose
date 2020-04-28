# ghost-docker-compose
Ghost blog using docker compose

Following the guide _"Hosting a Ghost blog with NGINX and Docker"_ from Alistair Chapman:

* Part 1: https://blog.agchapman.com/ghost-blog-with-nginx-and-docker-1/
* Part 2: https://blog.agchapman.com/ghost-blog-with-nginx-and-docker-2/

### Problems I came across

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
