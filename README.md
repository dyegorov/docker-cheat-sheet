# docker-cheat-sheet
## Container commands
### [docker run](https://docs.docker.com/engine/reference/commandline/run)
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
* `1.11` image version (also can be `latest`, `before` or some other tag)
* `--tty = -t` Allocates pseudo tty
* `-t` is used to change default command ran on container start

```
$ docker run --name mysql -e MYSQL_ROOT_PASSWORD=q1 --publish 3306:3306 -d mysql
```
* `--env = -e` Set env variables
