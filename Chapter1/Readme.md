# Chapter1 Getting Started with Docker

## Running Hello world in Docker

* run a container and echo "Hello world" in it:

  `docker run busybox echo hello world`

* list the running containers:
    
  `docker ps`

* list all the containers:
    
  `docker ps -a`

* list the images:

  `docker images`

* run a container and get a terminal session from it:

  `docker run -t -i ubuntu:14.04 /bin/bash`

## Running  Docker in detached mode

* run a container in detached mode:

```
docker create -P --expose=1234 python:2.7 python -m SimpleHTTPServer 1234
docker ps -a
docker start CONTAINERID
docker ps
```

## Building a Docker image with a Dockerfile

* see file **Dockerfile**

* build the new image called busybox2:

  `docker build -t busybox2 .`

* run a container using the new image:

  `docker run -i -t busybox2 sh`

## Using supervisor to run WordPress in a single container

* see file **Dockerfile_wp**, **wp-config.php** and **uspervisord.conf**

* build the wordpress image:

  `docker build -t wordpress .`

* run a container use the wp image:

  `docker run -d -p 80:80 wordpress`

## Running a wordpress blog using 2 linked containers

* start by pulling wordpress and mysql containers:

```
   docker pull wordpress:latest
   docker pull mysql:5.7
   docker images
 ```
* start a MySQL container:

   `docker run --name mysqlwp -e MYSQL_ROOT_PASSWORD=wordpressdocker -d mysql:5.7`

* start the wp container and link it the to MySQL one:

   `docker run --name wordpress --link mysqlwp:mysql -p 80:80 -d wordpress`

## Backing up a database running in a container

* start a container with bind mount on the host directory (/home/docker/mysql on /var/lib/mysql):

```
   docker run --name mysqlwp -e MYSQL_ROOT_PASSWORD=wordpressdocker \
                             -e MYSQL_DATABASE=wordpress \
                             -e MYSQL_USER=wordpress \
                             -e MYSQL_PASSWORD=wordpresspwd \
                             -v /home/docker/mysql:/var/lib/mysql \
                             -d mysql
```

## Sharing data in your Docker host with containers

* share the working directory of the host within a /cookbook directory in a container:

   `docker run -ti -v "$PWD":/cookbook debian:jessie /bin/bash`

* inspect the mount mapping with docker inspect command:

   `docker inspect -f {{.Mounts}} CONTAINER_ID`

## Sharing data between containers

* first, we create a data container that we will later share with another container:

   `docker run -v /data --name data ubuntu:14.04 /bin/bash`

  then, we create another container to who we mount the data container:

   `docker run -ti --volumes-from data ubuntu:14.04 /bin/bash`

## Copying data to and from containers

* first, we create a test container:

   `docker run -d --name testcopy ubuntu:14.04 sleep 360`

  then, inside the container, we create a file:

```
   docker exec -ti testcopy /bin/bash`
   cd /root
   echo 'I am in the container' > file.txt
   exit
```

  now, we retrieve the file with the docker cp command:

   `docker cp testcopy:/root/file.txt .`

  then, we update the file's content, and we copy it back on the conatainer:

```
   echo "I am now in the host" >> file.txt
   docker cp file.txt testcopy:/root/file.txt
```
