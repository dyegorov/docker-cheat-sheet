# docker-cheatsheet
* [Container](#container)
* [Image](#image)
* [Network](#network)
* [Volume](#volume)
* [Compose](#compose)
* [Docker machine](#docker-machine)
* [Swarm](#swarm)
* [Stacks](#stacks)
* [Secrets](#secrets)
* [Healthcheck](#healthcheck)
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
$ docker run --rm -d --name mongo mongo
```
* `--rm` Remove container when it exits

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

### Remove network
```
docker@node1:~$ docker network rm azo mln cs8 4rn wxq
```
Removes multiple networks

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

## Docker machine
### Create machine
```
$ docker-machine create node1
```
### List docker machines
```
$ docker-machine ls
```
### Connect to machine
```
$ docker-machine ssh node1
```
### View machine env
```
$ docker-machine env node1
```

## Swarm
* Cluster software built in in docker
* Node can be `manager` or `worker`. Managers are workers that can control swarm
* Manager nodes have RAFT db to store swarm configuration (root CA, configs and secrets)
### Initialize
Enables swarm (disabled by default)
```
$ docker swarm init
docker@node1:~$ docker swarm init --advertise-addr 192.168.99.100
```
Get manager token
```
docker@node1:~$ docker swarm join-token manager
To add a manager to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-1mgjen...1yztui 192.168.99.100:2377
```
### Join
```
docker@node2:~$ docker swarm join --token SWMTKN-1-1mgje...te7kd7 192.168.99.100:2377
```
Join as manager:
```
docker@node3:~$ docker swarm join --token SWMTKN-1-1mgjen...1yztui 192.168.99.100:2377
This node joined a swarm as a manager.
```
### Upgrade node from worker to manager
```
docker@node1:~$ docker node update --role manager node2
```
### Get node info
```
$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
agc1km66adr5yk119aoyc1djw *   node1               Ready               Active              Leader
og0d0omuncjvj3yzitul1sjsx     node2               Ready               Active              Reachable
c3nn53kc3wfyrfu24r99d796j     node3               Down                Active              Unreachable
$ docker node ps
```

### Run service
* Analogy: `docker service` = `docker run`
* Services are detached by default (use `detach=false` if needed)
```
$ docker service create alpine ping 8.8.8.8
docker@node1:~$ docker service create --replicas 3 alpine ping 8.8.8.8
```

### Service info
```
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
y7v0hwc1j0ao        brave_bose          replicated          1/1                 alpine:latest

$ docker service ps brave_bose
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
pa2siny46j1a        brave_bose.1        alpine:latest       moby                Running             Running about a minute ago
```
### Update service
* Is used to make rolling updates
* Limits downtime NOT prevents it
* Will replace containers for most changes
* Has a lot of options [docker service update](https://docs.docker.com/engine/reference/commandline/service_update/#options)
* Includes rollback and healthcheck options
* Scale and rollback quick access: `docker service scale web=4`, `docker service rollback web`
* `stack deploy` is update (or update is deploy). Just edit yml and stack deploy it.
```
$ docker service update y7v0hwc1j0ao --replicas 3
```
Update to newer image
```
docker service update --image myapp:1.2.1 <servicename>
```
Add env and remove port
```
docker service update --env-add NODE_ENV=production --publish-rm 8080
```
Change number of replicas of two services
```
docker service scale web=8 api=6
```

Example of updating. First change replicas:
```
docker@node1:~$ docker service create -p 8088:80 --name web nginx:1.13.7
docker@node1:~$ docker service scale web=3
```
Next update
```
docker@node1:~$ docker service update --image nginx:1.13.8 web
```
By default updates are sync (updates first, verifies, updates second etc.)

Change port
```
docker@node1:~$ docker service update --publish-rm 8088 --publish-add 9090:80 web
```

For rebalancing purpose u may need just update - this will reboot nodes and rebalance them
```
docker@node1:~$ docker service update --force web
```

### Remove service
```
$ docker service rm brave_bose
```
### Overlay multihost networking
* Overlay network - the only we can use in our swarm. It spans accross nodes.
* Just choose --driver overlay when creating network
* For container-to-container traffic inside a single Swarm
* Optional IPSec (AES) encryption on network creation (disabled by default - performance)
* Each service can be connected to multiple networks (e.g. front-end, back-end)

Example
```
docker@node1:~$ docker service create --name psql --network mydrupal -e POSTGRES_PASSWORD=mypass postgres
211xugmruk5w66vrc8ter7fmp
overall progress: 1 out of 1 tasks
1/1: running   [==================================================>]
verify: Service converged

docker@node1:~$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
7f52yjt5gbv9        lucid_lumiere       replicated          3/3                 alpine:latest
211xugmruk5w        psql                replicated          1/1                 postgres:latest

docker@node1:~$ docker service ps psql
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                    ERROR               PORTS
wwqlahql9t6c        psql.1              postgres:latest     node2               Running             Running less than a second ago

docker@node1:~$ docker service create --name drupal --network mydrupal -p 80:80 drupal
y6vwiscda8mbcph4bier5m39o
overall progress: 1 out of 1 tasks
1/1: running   [==================================================>]
verify: Service converged
```
### Routing Mesh
* Routing ingress (incoming) packets for a service to proper task
* Spans all nodes in swarm
* Users IPVS from Linux Kernel
* Load balances Swarm Services across their tasks
* 2 ways this works:
  * Container-to-container in overlay network (uses VIP)
  * External traffic incoming to published ports (all nodes listen)

### Routing Mesh Cont
* This is stateless load balancing
* This LB is at OSI layer 3 (TCP), not L4 (DNS)
Both limitations can be overcome with:
* Nginx or HAProxy LB proxy
* Docker Enterprise Edition (comes with L4 web proxy)

## Stacks
* Stacks accept compose files as their definition for services, netwroks, and volumes
* We use `docker stack deploy` rather then `docker service create`
* Stacks manages all those objects for us, including overlay network per stack. Adds stack name to start of their name
* New `deploy:` key in Compose file. Can't do `build:`
* Compose now ignores `deploy:`, Swarm ignores `build:`
* `docker-compose` cli not needed on swarm server
* Stack is only for ONE swarm

`docker stack deploy` also updates services
```
docker@node1:~$ cat example.yml
version: '3'
services:
    redis:
        image: redis:alpine
        networks:
            - frontend
    db:
        image: postgres:9.6
        volumes:
            - db-data:/var/lib/postgresql/data
        networks:
            - backend
        deploy:
    vote:
        image: bretfisher/examplevotingapp_vote
        ports:
            - '5000:80'
        networks:
            - frontend
        deploy:
            mode: global
    result:
        image: bretfisher/examplevotingapp_result
        ports:
            - '5001:80'
        networks:
            - backend
    worker:
        image: bretfisher/examplevotingapp_worker:java
        networks:
            - frontend
            - backend
        deploy:
            mode: global
networks:
    frontend:
    backend:
volumes:
    db-data:
```
This file is run via:
```
docker@node1:~$ docker stack deploy -c example.yml voteapp
```

And here is cli analog:
```
docker@node1:~$ docker network create -d overlay backend
docker@node1:~$ docker network create -d overlay frontend

docker@node1:~$ docker service create --name vote -p 80:80 --network frontend --replicas 2 dockersamples/examplevotingapp_vote

docker@node1:~$ docker service create --name redis --network frontend --replicas 2 redis:3.2

docker@node1:~$ docker service create --name worker --network frontend --network backend dockersamples/examplevotingapp_worker

docker@node1:~$ docker service create --name db --network backend --mount type=volume,source=db-data,target=/var/lib/postgresql/data postgres:9.4

docker@node1:~$ docker service create --name result --network backend -p 5001:80 docker
samples/examplevotingapp_result
```

### Stack info
List stacks
```
docker@node1:~$ docker stack ls
NAME                SERVICES
mydb                1
```
View tasks
```
docker@node1:~$ docker stack ps voteapp
```
View services
```
docker@node1:~$ docker stack services voteapp
```

### Remove stack
```
docker@node1:~$ docker stack rm mydb
Removing service mydb_psql
Removing secret mydb_psql_user
Removing secret mydb_psql_password
Removing network mydb_default
```

## Secrets
* Raft db is encrypted on disk
* Secrets stored on disk on Manager nodes
* Default is Managers and Workers "control plane" TLS + Mutual auth
* Secrets are stored in swarm and only then assigned to services
* Only containers in assigned services can see them
* They look like files in container but are in-memory fs (`/run/sercets/secret_name_or_alias`)

### Create secret
```
docker@node1:~$ echo mypsqluser >> psql_user.txt
docker@node1:~$ cat psql_user.txt
mypsqluser
docker@node1:~$ docker secret create psql_user psql_user.txt
m25s6vxerqes5oprpgxk8gvhc
docker@node1:~$ echo DBpassword | docker secret create psql_pass -
wpzrdsrdsep2ol4h5img5cz7b
```
`-` at the end means 'read from std input'

2 security concerns:
* 1st way - password is stored in a file
* 2nd - password is stored in `history`

howto clean your history
```
$ echo saved
saved
```
prepend with space 
```
$  echo not saved
not saved
```
or remove line
```
$ history -d 2010
```
[view on stackoverflow](https://unix.stackexchange.com/questions/49214/how-to-remove-a-single-line-from-history)

### View secrets
```
docker@node1:~$ docker secret ls
ID                          NAME                DRIVER              CREATED             UPDATED
adhcfgnlgahof01zhndxyjzrv   psql_pass                               1 second ago        1 second ago
m25s6vxerqes5oprpgxk8gvhc   psql_user                               10 minutes ago      10 minutes ago
```

```
docker@node1:~$ docker secret inspect psql_user
[
    {
        "ID": "m25s6vxerqes5oprpgxk8gvhc",
        "Version": {
            "Index": 2577
        },
        "CreatedAt": "2018-01-30T12:23:01.986114529Z",
        "UpdatedAt": "2018-01-30T12:23:01.986114529Z",
        "Spec": {
            "Name": "psql_user",
            "Labels": {}
        }
    }
]
```

### Remove secret
```
docker@node1:~$ docker secret rm wpz
```

### Use secret
```
docker@node1:~$ docker service create --name psql --secret psql_user --secret psql_pass 
-e POSTGRES_PASSWORD_FILE=/run/secrets/psql_pass
-e POSTGRES_USER_FILE=/run/secrets/psql_user postgres
```
* `--secret` maps secret to container
* `-e POSTGRES_PASSWORD_FILE=/run/secrets/psql_pass` here `_FILE` reads file and stores its contents in envdo

Secrets are stored in memory:
```
docker@node1:~$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
93ptf7b7tzzc        psql                replicated          1/1                 postgres:latest
docker@node1:~$ docker service ps psql --no-trunc
ID                          NAME                IMAGE  NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
jrrc3y0bpa3ex89brft6w5t4r   psql.1              postgres:latest@sha256:3f4441460029e12905a5d447a3549ae2ac13323d045391b0cb0cf8b48ea17463  node1               Running             Running 4 minutes ago

docker@node1:~$ docker exec -it psql.1.jrrc3y0bpa3ex89brft6w5t4r bash

root@716f494ad35e:/# ls /run/secrets
psql_pass  psql_user
root@716f494ad35e:/# cat /run/secrets/psql_user
mypsqluser
```

### Remove secret from service
When you do it - container will be recreated (because of immutability)
```
docker@node1:~$ docker service update --secret-rm
```

### Secrets and stacks
```
$ ls
docker-compose.yml      psql_password.txt       psql_user.txt

$ cat docker-compose.yml
version: "3.1"

services:
  psql:
    image: postgres
    secrets:
      - psql_user
      - psql_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/psql_password
      POSTGRES_USER_FILE: /run/secrets/psql_user

secrets:
  psql_user:
    file: ./psql_user.txt
  psql_password:
    file: ./psql_password.txt
```

```
docker@node1:~$ docker stack deploy -c example.yml mydb
docker@node1:~$ docker secret ls
ID                          NAME                 DRIVER              CREATED             UPDATED
itasfav4hsegzvxy7ei8dvrxk   mydb_psql_password                       46 seconds ago      46 seconds ago
882dqicp3grqqqbu6l4bcq5ia   mydb_psql_user                           46 seconds ago      46 seconds agodo
```

Another stack secret example
```
docker@node1:~$ cat example.yml
version: '3.1'

services:
  drupal:
    image: drupal:8.2
    ports:
      - "8080:80"
    volumes:
      - drupal-modules:/var/www/html/modules
      - drupal-profiles:/var/www/html/profiles
      - drupal-sites:/var/www/html/sites
      - drupal-themes:/var/www/html/themes

  postgres:
    image: postgres:9.6
    environment:
      - POSTGRESS_PASSWORD_FILE=/run/secrets/psql-pw
    secrets:
      - psql-pw
    volumes:
      - drupal-data:/var/lib/postgresql/data


volumes:
  drupal-data:
  drupal-modules:
  drupal-profiles:
  drupal-sites:
  drupal-themes:

secrets:
  psql-pw:
    external: true
```
```
docker@node1:~$ echo "dakKrtnd" | docker secret create psql-pw -
5kjgzst3jbd7ux2cxvhmrbyhg
docker@node1:~$ docker stack deploy -c example.yml drupal
Creating network drupal_default
Creating service drupal_drupal
Creating service drupal_postgres
```

### Secrets and compose (for dev purposes)
Compose works only with files (NO external secrets)
```
docker@node1:~$ cat docker-compose.yml
version: "3.1"

services:
  psql:
    image: postgres
    secrets:
      - psql_user
      - psql_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/psql_password
      POSTGRES_USER_FILE: /run/secrets/psql_user

secrets:
  psql_user:
    file: ./psql_user.txt
  psql_password:
    file: ./psql_password.txt

$ docker-compose up -d

$ docker-compose exec psql cat /run/secrets/psql_user
dbuser
```

## Healthcheck
* Executes commands in container (e.g. `curl localhost`)
* It expects `exit 0` (OK) or `exit 1` (Error)
* Three container states: starting, healthy, unhealthy
* Better then "is binary still running"
* Not replacing monitoring

### Example with docker run
```
$ docker run \
> --health-cmd="curl -f localhost:9200/_cluster/health || false" \
> --health-interval=5s \
> --health-retries=3 \
> --health-timeout=2s \
> --health-start-period=15s \
> elasticsearch:2
```
```
$ docker container run --name p1 -d postgres
$ docker container run --name p2 -d --health-cmd="pg_isready -U postgres || exit 1" postgres

$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED                  STATUS                           PORTS               NAMES
69d60ec8dd0e        postgres            "docker-entrypoint.s…"   Less than a second ago   Up 1 second (health: starting)   5432/tcp            p2
11f5b479bd90        postgres            "docker-entrypoint.s…"   51 seconds ago           Up 59 seconds                    5432/tcp            p1

$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS                    PORTS               NAMES
69d60ec8dd0e        postgres            "docker-entrypoint.s…"   33 seconds ago       Up 40 seconds (healthy)   5432/tcp            p2
11f5b479bd90        postgres            "docker-entrypoint.s…"   About a minute ago   Up About a minute         5432/tcp            p1
```

### Healthcheck in dockerfile
```
HEALTHCHECK curl -f http://localhost/ || false
or
HEALTHCHECK --timeout=2s --interval=3s --retries=3 \
  CMD curl -f http://localhost/ || exit 1
```
`exit 1` and `false` is the same thing
```
FROM nginx:1.13

HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
```
```
FROM postgres
HEALTHCHECK --interval=5s --timeout=3s \
  CMD pg_isready -U postgres || exit 1
```
### Healthcheck in compose/stack files
```
version: "3.4"
services:
  web:
    image: nginx
    healthcheck:
      test: ["CMD", "curl", "-f","http://localhost"]
      interval: 1m30s
      timeout: 10s
      retries: 3
      start_period: 1m
```

## Container registry
hub.docker.com

TODO: Automated builds

store.docker.com

cloud.docker.com

## Examples
### Multiple environments orchestration
```
$ ls
Dockerfile			docker-compose.prod.yml		docker-compose.yml		sample-data
docker-compose.override.yml	docker-compose.test.yml		psql-fake-password.txt
```
```
$ cat docker-compose.yml
version: '3.1'

services:

  drupal:
    image: bretfisher/custom-drupal:latest

  postgres:
    image: postgres:9.6
```
`docker-compose.yml` is run by default

```
$ cat docker-compose.override.yml
version: '3.1'

services:

  drupal:
    build: .
    ports:
      - "8080:80"
    volumes:
      - drupal-modules:/var/www/html/modules
      - drupal-profiles:/var/www/html/profiles
      - drupal-sites:/var/www/html/sites
      - ./themes:/var/www/html/themes

  postgres:
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/psql-pw
    secrets:
      - psql-pw
    volumes:
      - drupal-data:/var/lib/postgresql/data

volumes:
  drupal-data:
  drupal-modules:
  drupal-profiles:
  drupal-sites:
  drupal-themes:

secrets:
  psql-pw:
    file: psql-fake-password.txt
```
`docker-compose.override.yml` runs after `docker-compose.yml` and overrides it automatically

```
$ docker-compose up -d
$ docker container inspect swarmstack3_drupal_1
...
"Type": "volume",
"Name": "swarmstack3_drupal-profiles",
...
```
Volumes from override config are mounted.

Now lets run test env
```
$ cat docker-compose.test.yml
version: '3.1'

services:

  drupal:
    image: bretfisher/custom-drupal
    build: .
    ports:
      - "80:80"

  postgres:
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/psql-pw
    secrets:
      - psql-pw
    volumes:
      - ./sample-data:/var/lib/postgresql/data
secrets:
  psql-pw:
    file: psql-fake-password.txt
```
```
$ docker-compose -f docker-compose.yml -f docker-compose.test.yml up -d
$ docker container inspect swarmstack3_drupal_1
...
"Mounts": [],
...
```

Lets run production env
```
$ cat docker-compose.prod.yml
version: '3.1'

services:

  drupal:
    ports:
      - "80:80"
    volumes:
      - drupal-modules:/var/www/html/modules
      - drupal-profiles:/var/www/html/profiles
      - drupal-sites:/var/www/html/sites
      - drupal-themes:/var/www/html/themes

  postgres:
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/psql-pw
    secrets:
      - psql-pw
    volumes:
      - drupal-data:/var/lib/postgresql/data

volumes:
  drupal-data:
  drupal-modules:
  drupal-profiles:
  drupal-sites:
  drupal-themes:

secrets:
  psql-pw:
    external: true
```
View combined config
```
$ docker-compose -f docker-compose.yml -f docker-compose.prod.yml config
$ docker-compose -f docker-compose.yml -f docker-compose.prod.yml config > output.yml
```

### Mongo db and mongo express in stack
```
$ cat mongo.yml
version: '3'
services:
  mongo:
    image: mongo:3.6.2
    volumes:
      - db-data:/data/db
    networks:
      - backend
  admin:
    image: mongo-express:0.42.2
    ports:
      - "8081:8081"
    networks:
      - backend
    depends_on:
      - db
networks:
  backend:
volumes:
  db-data:
```
```
$ docker stack deploy -c mongo.yml mongo
```

---
## Links
* [Official docs](https://docs.docker.com/edge/engine/reference/commandline/docker/)
* [Someone's cheatsheet](https://github.com/wsargent/docker-cheat-sheet/blob/master/README.md)
* [Docker mastery src](https://github.com/BretFisher/udemy-docker-mastery)
