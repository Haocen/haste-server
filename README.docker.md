# Hastebin

[Hastebin](https://github.com/seejohnrun/haste-server) is a simple
pastebin, which can be installed on a protected network.

This dockerfile builds an image that can be configured using
environment variables. This is done by writing `config.js` at runtime
from interpolated variables in `app.sh`.

See `app.sh` for variable names.

## Example use

> `STORAGE_TYPE` is set to `file` by default so you can quickly be up and running!

Writing pastes to a mounted local volume:

```
docker run --name hastebin -d -p 7777:7777 -e STORAGE_TYPE=file -v /data:/app/data rlister/hastebin
```

Writing pastes to redis:

```
docker run --name redis -d redis
docker run --name hastebin -d -p 7777:7777 --link redis:redis -e STORAGE_TYPE=redis -e STORAGE_HOST=redis rlister/hastebin
```

## Example docker compose

```
version: '3'

services:
   db:
     image: redis:4
     container_name: hastebin_redis
     volumes:
       - /var/home/user/hastebin-redis:/data:Z
     restart: unless-stopped
     networks:
       - hastebin_internal
     command: redis-server --appendonly yes

   hastebin:
     depends_on:
       - db
     image: haocen/haste-server
     container_name: hastebin_server
     links:
       - db:redisDb
     networks:
       - hastebin_internal
     ports:
       - "7777:7777"
     restart: unless-stopped
     environment:
       STORAGE_TYPE: redis
       STORAGE_HOST: redisDb
       STORAGE_PORT: 6379
       STORAGE_EXPIRE: 2592000
networks:
    hastebin_internal:
      driver: bridge
```