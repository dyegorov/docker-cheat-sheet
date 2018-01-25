# docker-cheat-sheet
## Container commands
### Run new container 
[docker run](https://docs.docker.com/engine/reference/commandline/run) (docker container run)
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

### Run shell and connect to running container
```
$ docker container exec -it mongo bash
root@73f330790898:/#
```
Runs `bash` in RUNNING container

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

## Image commands
### Download image
```
$ docker pull alpine
```
### View downloaded images
```
$ docker image ls
```
## Network commands
Docker networks Defaults
* Each container connected to a private virtual network "bridge"
* Each virtual network routes through NAT firewall on host IP
* All container on a virtual network can talk to each other without -p
* Best practice is to create a new virtual netwrok for each app
  * network "my_web_app" for frontend
  * network "my_api" for api/backend and db