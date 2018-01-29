# docker-cheat-sheet
* [Container](#container)
* [Image](#image)
* [Network](#network)
* [Volume](#volume)
* [Compose](#compose)
* [Swarm](#swarm)
* [Links](#links)
## Container
### Run new container 
[docker run](https://docs.docker.com/engine/reference/commandline/run) (`docker container run` is synonym i think)
```
$ docker container run --publish 80:80 nginx
```
1. Downloads image nginx from hub.docker.com
2. Starts new container from that image
3. Openes port 80 on host
4. Routes traffic host:80 => container:80

```
$ docker container run --publish 80:80 --detach nginx
$ docker container run -p 80:80 -d nginx
```
* `--detach = -d` Runs container in background and prints its ID
* `--publish = -p` Publish container's port to host

```
$ docker container run -p 8080:80 -d --name webhost nginx:1.11 -t
```
* Assigns `webhost` name to container
* `8080` hosts listening port
* `1.11` image version (also can be `alpine`, `latest`, `before` or some other tag)
* `--tty = -t` Allocates pseudo tty
* `-t` is used to change default command ran on container start

```
$ docker run --name mysql -e MYSQL_ROOT_PASSWORD=q1 --publish 3306:3306 -d mysql
```
* `--env = -e` Set env variables

```
$ docker run --name mongo -d mongo
```
Run a command in a new container

```
$ docker container run -it --name proxy nginx bash
```
* `--interactive = -i` 
Starts new container and runs `bash` in it
```
$ docker run -rm nginx
```
* `-rm` Remove container when it exits

```
$ docker container run -d --name mysql2 -e MYSQL_ALLOW_EMPTY_PASSWORD=true -v mysql-db:/var/lib/mysql mysql
```
Runs container with volume
* `--volume = -v` mounts volume

### Use running container
```
$ docker container exec -it mongo bash
root@73f330790898:/#
```
Runs `bash` in RUNNING container

```
$ docker attach www
```
Attach stdi, stdo, stderr to container
> You can detach from container and leave it running with
> `CTRL-p CTRL-q` (from docs, i failed to do so and don't know use case for this)

### Get container info
```
$ docker ps
$ docker container ls
```
View running containers

```
$ docker ps -a
$ docker container ls -a
```
View ALL containers (running and stopped)
```
$ docker container logs webhost
```
View container logs

```
$ docker container top webhost
```
View ONE container processes

```
$ docker container stats
$ docker stats
```
Performance stats for ALL containers

```
$ docker container inspect www
```
View container config
```
$ docker container port www
```
View port forwarding info
```
$ docker container run -d --name psql9 -v psql:/var/lib/postgresql/data postgres:9
$ docker container logs -f psql9
```
* `-f = follow` to view container logs in realtime

### Stop container
```
$ docker stop www
$ docker stop www mongo
$ docker container stop 87f
$ docker container stop -t 10 www mongo
```
* Stop one o more containers (using SIGTEM)
* `--time = -t` Timeout in seconds

```
docker kill www
```
Stop container(s) using SIGKILL

### Start container
```
$ docker container start www mongo
$ docker start www mongo
```

### Remove container
```
$ docker container rm a69 87f
$ docker container rm www
```
Removes stopped container(s)
```
$ docker container rm -f www mongo
```
Removes all containers (even running)
```
$ docker run -rm nginx
```
* `-rm` Remove container when it exits

```
$ docker rm -f $(docker ps -aq)
```
Delete all running and stopped containers

## Image
### Download image
```
$ docker pull alpine
```
### View downloaded images
```
$ docker image ls
```
### View how image was built
```
$ docker history nginx
```
### Tag image
```
$ docker image tag nginx myname/ngynx
$ docker image tag myname/nginx testing
```
### Push image to hub
```
$ docker image push myname/ngynx
```
### Login/logout
```
$ docker login
```

```
$ cat .docker/config.json
{
  "auths" : {

  }
}
```

```
$ docker logout
```
### Build image
```
$ docker image build -t customnginx .
```

Dockerfile:
```
# - you should use the 'node' official image, with the alpine 6.x branch
FROM node:6-alpine
# - this app listens on port 3000, but the container should launch on port 80
# so it will respond to http://localhost:80 on your computer
EXPOSE 3000
# - then it should use alpine package manager to install tini: 'apk add --update tini'
RUN apk add --update tini
# - then it should create directory /usr/src/app for app files with 'mkdir -p /usr/src/app'
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
# - Node uses a "package manager", so it needs to copy in package.json file
COPY package.json package.json
# - then it needs to run 'npm install' to install dependencies from that file
# - to keep it clean and small, run 'npm cache clean --force' after above
RUN npm install && npm cache clean --force
# - then it needs to copy in all files from current directory
COPY . .
# - then it needs to start container with command 'tini -- node ./bin/www'
CMD ["tini", "--", "node", "./bin/www"]
```
### Remove image from local image store
```
$ docker rmi alpine:3.4
```
## Network
### Docker networks defaults
* Each container connected to a private virtual network "bridge"
* Each virtual network routes through NAT firewall on host IP
* All container on a virtual network can talk to each other without -p
* Best practice is to create a new virtual netwrok for each app
  * network "my_web_app" for frontend
  * network "my_api" for api/backend and db

### Default docker drivers
* bridge (`--network` option not specified). Default `docker0` network built in all installations.
* host (`--network host`). Connects to host directly, skips virtual networks. Less security, more performance.
* none (`--network none`). No network interface

### Docker Networks: Default Security
* Create your apps so frontend/backend sit on same Docker network
* Their inter-communication never leaves host
* All externally exposed ports closed by default
* You must. manually expose via -p, which is better default security
* This get even better later with Swarm and Overlay networks
	
### Docker DNS
Do not rely on IP's - use docker names. DNS is built in. Always create custom networks. Default bridge network doesn't have DNS(((

### Network info
```
$ docker container port www
```
View port forwarding info
```
$ docker container inspect --format '{{.NetworkSettings.IPAddress}}' www
```
View container ip

```
$ docker network ls
```
List networks

```
$ docker network inspect host
```
View network config

### Create network
```
$ docker network create my_app_net
```
Creating network using default driver `bridge`

### Connect container to network
```
$ docker network connect my_app_net webhost
```

## Volume
### Volume info
```
$ docker volume ls
```
Lists volumes

```
$ docker volume inspect ab75a759d583a2906fc183c07a3ba9f44e43f6720df52c03175172a199b01436
```
View volume config

```
$ docker container run -d --name mysql2 -e MYSQL_ALLOW_EMPTY_PASSWORD=true -v mysql-db:/var/lib/mysql mysql
$ docker container run -d --name nginx -p 80:80 -v $(pwd):/usr/share/nginx/html nginx
$ docker container run -d --name psql9 -v psql:/var/lib/postgresql/data postgres:9
```
Runs container with volume
* `--volume = -v` mounts volume

### Remove all unused volumes
```
$ docker volume prune
```

## Compose

Compose is NOT a production-grade tool but good for local development and test

* Configure relationships between containers
* Save docker container run settings in easy-to-read file
* Create one-liner developer environment startups
* Consists of 2 things:
  * YAML formatted file `docker-compose.yml` describing options for containers, networks, volumes
  * CLI tool `docker-compose` used for loacl dev/test automation with those yaml files

Now can be used directly with Swarm. `docker-compose.yml` is default filename, but ane can be used with `docker-compose -f`

Compose file template
```
version: '3.1'  # if no version is specificed then v1 is assumed. Recommend v2 minimum. Use v3)))

services:  # containers. same as docker run
  servicename: # a friendly name. this is also DNS name inside network
    image: # Optional if you use build:
    command: # Optional, replace the default CMD specified by the image
    environment: # Optional, same as -e in docker run
    volumes: # Optional, same as -v in docker run
  servicename2:

volumes: # Optional, same as docker volume create

networks: # Optional, same as docker network create
```

Examples of compose files (compose.yml)
```
version: '2'

# same as 
# docker run -p 80:4000 -v $(pwd):/site bretfisher/jekyll-serve

services:
  jekyll:
    image: bretfisher/jekyll-serve
    volumes:
      - .:/site
    ports:
      - '80:4000'

```

```
version: '2'

services:

  wordpress:
    image: wordpress
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: example
      WORDPRESS_DB_PASSWORD: examplePW
    volumes:
      - ./wordpress-data:/var/www/html

  mysql:
    image: mariadb
    environment:
      MYSQL_ROOT_PASSWORD: examplerootPW
      MYSQL_DATABASE: wordpress
      MYSQL_USER: example
      MYSQL_PASSWORD: examplePW
    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  mysql-data:
```
```
version: '3'

services:
  ghost:
    image: ghost
    ports:
      - "80:2368"
    environment:
      - URL=http://localhost
      - NODE_ENV=production
      - MYSQL_HOST=mysql-primary
      - MYSQL_PASSWORD=mypass
      - MYSQL_DATABASE=ghost
    volumes:
      - ./config.js:/var/lib/ghost/config.js
    depends_on:
      - mysql-primary
      - mysql-secondary
  proxysql:
    image: percona/proxysql
    environment: 
      - CLUSTER_NAME=mycluster
      - CLUSTER_JOIN=mysql-primary,mysql-secondary
      - MYSQL_ROOT_PASSWORD=mypass
      - MYSQL_PROXY_USER=proxyuser
      - MYSQL_PROXY_PASSWORD=s3cret
  mysql-primary:
    image: percona/percona-xtradb-cluster:5.7
    environment: 
      - CLUSTER_NAME=mycluster
      - MYSQL_ROOT_PASSWORD=mypass
      - MYSQL_DATABASE=ghost
      - MYSQL_PROXY_USER=proxyuser
      - MYSQL_PROXY_PASSWORD=s3cret
  mysql-secondary:
    image: percona/percona-xtradb-cluster:5.7
    environment: 
      - CLUSTER_NAME=mycluster
      - MYSQL_ROOT_PASSWORD=mypass
      - CLUSTER_JOIN=mysql-primary
      - MYSQL_PROXY_USER=proxyuser
      - MYSQL_PROXY_PASSWORD=s3cret
    depends_on:
      - mysql-primary
```

### Start containers from compose
```
$ docker-compose up
$ docker-compose up -d
```
* `-d` detaches from containers

### View info
```
$ docker-compose ps
```
Shows running containers

```
$ docker-compose top
```
Shows container processes grouped by container

### Stop containers
```
$ docker-compose down
$ docker-compose down -v
$ docker-compose down --rmi local
```
* `-v = --volumes` removes named volumes declared in compose file
* `--rmi type` removes images. Type can be: `all` or `local`
Stops containers and removes containers, networks, volumes and images created by `up`

### Using compose to build
* Compose can also build custom images
* Will build them with `docker-compose up` if not found in cache
* Also rebuild with `docker-compose build`
* Good for complex builds that have lots of vars or build args

Example:
```
# docker-compose.yml
version: '2'

# based off compose-sample-2, only we build nginx.conf into image
# uses sample site from https://startbootstrap.com/template-overviews/agency/

services:
  proxy:
    build:
      context: .
      dockerfile: nginx.Dockerfile
    image: nginx-custom
    ports:
      - '80:80'
  web:
    image: httpd
    volumes:
      - ./html:/usr/local/apache2/htdocs/
```
Will `build` if can not find `nginx-custom` image
```
# nginx.Dockerfile
FROM nginx:1.11

COPY nginx.conf /etc/nginx/conf.d/default.conf
```
```
# nginx.conf
	listen 80;

	location / {

		proxy_pass         http://web;
		proxy_redirect     off;
		proxy_set_header   Host $host;
		proxy_set_header   X-Real-IP $remote_addr;
		proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header   X-Forwarded-Host $server_name;

	}
}
```

## Swarm
### Initialize
```
$ docker swarm init
```

### Get node info
```
$ docker node ls
```

### Run service
Analogy: `docker service` = `docker run`

```

```
## Links
* [Official docs](https://docs.docker.com/edge/engine/reference/commandline/docker/)
* [Someone's cheatsheet](https://github.com/wsargent/docker-cheat-sheet/blob/master/README.md)
* [Docker mastery src](https://github.com/BretFisher/udemy-docker-mastery)