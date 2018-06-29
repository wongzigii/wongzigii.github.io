title: Dancing with docker
date: 2016-08-08 22:22:48
tags: docker
---

### Create a machine
````bash
docker-machine create --driver virtualbox default
````
  
这里 default 是这个 docker machine 的名字。

### List available machines

````bash
docker-machine ls
````
 
NAME |     ACTIVE |  DRIVER  |     STATE |    URL                    |                    SWARM |  DOCKER |  ERRORS|
-------|------------|----------|-----------|---------------------------|--------------------------|---------|--------|
default|   -        |virtualbox|   Running |  tcp://192.168.99.100:2376|                          | v1.12.0 |        | 
toolbox|   -        |virtualbox|   Running |  tcp://192.168.99.101:2376|                          | Unknown |  Unable to query docker version: Get https://192.168.99.101:2376/v1.15/version: x509: certificate is valid for 192.168.99.100, not 192.168.99.101|

发现第二次报错：

> Unable to query docker version: Get https://192.168.99.101:2376/v1.15/version: x509: certificate is valid for 192.168.99.100, not 192.168.99.101

[解决办法](https://github.com/docker/machine/issues/531#issuecomment-168356493)：

````bash
docker-machine regenerate-certs default
````

### Connect your shell to the new machine

````bash
eval "$(docker-machine env default)"
````
 
如果不执行这个命令，会提示 `docker-machine` not found。  
    
### Get the host IP address

````bash
docker-machine ip default
````
    
> 192.168.99.100

### SSH connection

````bash
docker-machine ssh default
````
        
### Download and run MySQL

如果本地没有合适的镜像，则会从 Docker-Hub 的仓库拉取，-p 参数指定映射的端口。

````bash
docker run -e MYSQL_ROOT_PASSWORD=qweasd --name mysql -d -p=3306:3306 mysql    
````

如果镜像已经在运行的状态下再 `docker run foo` 会报错：

> Error response from daemon: Conflict. The name "foo" is already in use by container f9e5798a82e0. You have to delete (or rename) that container to be able to reuse that name.

请注意：

`docker run` 会新创建一个镜像

而

`docker start` 只会将（已存在）停止的镜像重新启动。

### Remove a docker

````bash
docker rm mysql    
````
    
### Start an existed docker

````bash
docker start mysql
````
    
### Stop a docker

````bash
docker stop mysql    
````
 
### List runing dockers

````bash
docker ps    
````
>    
|CONTAINER ID    |    IMAGE                        | COMMAND              |  CREATED         |       STATUS    |    PORTS                     |  NAMES  |
-----------------|---------------------------------|----------------------|------------------|-----------------|------------------------------|---------|
83bcb5091ba1     |   mysql                         |"docker-entrypoint.sh"| 55 minutes ago   |   Up 28 minutes | 0.0.0.0:3306->3306/tcp       |  mysql  |
cd232a5729da     |   daocloud.io/daocloud/daomonit |"/usr/local/bin/daomo"| About an hour ago|   Up 46 minutes |                              | daomonit|

`docker ps -a` 可以显示全部的镜像。

### Docker help

````bash
docker run --help    
````

### Links

[https://github.com/docker-library/mysql/issues/95](https://github.com/docker-library/mysql/issues/95)

[https://hub.docker.com/r/mysql/mysql-server/](https://hub.docker.com/r/mysql/mysql-server/)

[https://docs.docker.com/machine/get-started/](https://docs.docker.com/machine/get-started/)