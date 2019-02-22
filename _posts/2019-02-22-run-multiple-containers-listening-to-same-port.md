---
layout:         post
title:          How to run multiple docker containers listening to the same port
description:    Let's say you need to run multiple dockerized web servers on the same machine. Obviously you could just use a separate 
                port for each one, but we want to work with hostnames instead.
date:           2019-02-22 13:00:00 +0700
categories:     [docker]
---

Let's say you need to run multiple dockerized web servers on the same machine. Obviously you could just use a separate 
port for each one, but we want to work with hostnames instead.

There is no way around running a reverse proxy. Luckily enough that's not a big deal with Docker. There is an image just
for that. Add the following service to your `docker-compose.yml`:

```yml
services:

    nginx-proxy:
        image: jwilder/nginx-proxy
        ports:
          - "80:80"
        volumes:
          - /var/run/docker.sock:/tmp/docker.sock:ro
```

Now let's just add two different simple web services both listening to port 80 to test the proxy:

```yml
services:

    nginx-proxy:
        # ...

    web1:
        image: tutum/hello-world
        environment:
          - VIRTUAL_HOST=web1.local
    
    web2:
        image: jwilder/whoami
        environment:
          - VIRTUAL_HOST=web2.local
```

Finally, map the hostnames to localhost in your `/etc/hosts` file. Just add the following lines:

```
127.0.0.1   web1.local
127.0.0.1   web2.local
```

**That's it!** Now do a `docker-compose up` and and access the services with your browser. You should see the following:

| [web1.local](http://web1.local)                                                                 | [web2.local](http://web1.local)                                                                 |
| ----------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| ![web1.local](/assets/posts/2019-02-22-run-multiple-containers-listening-to-same-port/web1.png) | ![web2.local](/assets/posts/2019-02-22-run-multiple-containers-listening-to-same-port/web2.png) |


## Links
* [My final docker-compose.yml](/assets/posts/2019-02-22-run-multiple-containers-listening-to-same-port/docker-compose.yml)
* [Nginx Proxy on Docker Hub](https://hub.docker.com/r/jwilder/nginx-proxy)