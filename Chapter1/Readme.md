# Chapter1 Getting Started with Docker

## Running Hello world in Docker

* Run a container and echo "Hello world" in it:

  `docker run busybox echo hello world`

* List the running Containers:
    
  `docker ps`

* List all the Containers:
    
  `docker ps -a`

* List the images:

  `docker images`

* Run a container and get a terminal session from it:

  `docker run -t -i ubuntu:14.04 /bin/bash`

## Running  Docker in detached mode

* Run a container in detached mode:

```
docker create -P --expose=1234 python:2.7 python -m SimpleHTTPServer 1234
docker ps -a
docker start CONTAINERID
docker ps
```

## Building a Docker image with a Dockerfile

* see file **Dockerfile**

* Build the new image called busybox2:

  `docker build -t busybox2 .`

* Run a container using the new image:

  `docker run -i -t busybox2 sh`

## Using supervisor to run WordPress in a single container

* see file **Dockerfile_wp**, **wp-config.php** and **uspervisord.conf**

* Build the wordpress image:

  `docker build -t wordpress .`

* Run a container use the wp image:

  `docker run -d -p 80:80 wordpress`

## Running a wordpress blog using 2 linked containers

* Start by pulling wordpress and mysql containers:

```
   docker pull wordpress:latest
   docker pull mysql:5.7
   docker images
 ```
* Start a MySQL container:

   `docker run --name mysqlwp -e MYSQL_ROOT_PASSWORD=wordpressdocker -d mysql:5.7`

* Start the wp container and link it the to MySQL one:

   `docker run --name wordpress --link mysqlwp:mysql -p 80:80 -d wordpress`

## Backing up a database running in a container

* Start a container with bind mount on the host directory (/home/docker/mysql on /var/lib/mysql):

```
   docker run --name mysqlwp -e MYSQL_ROOT_PASSWORD=wordpressdocker \
                             -e MYSQL_DATABASE=wordpress \
                             -e MYSQL_USER=wordpress \
                             -e MYSQL_PASSWORD=wordpresspwd \
                             -v /home/docker/mysql:/var/lib/mysql \
                             -d mysql
```

## Sharing data in your Docker host with containers

* Share the working directory of the host within a /cookbook directory in a container:

   `docker run -ti -v "$PWD":/cookbook debian:jessie /bin/bash`

* Inspect the mount mapping with docker inspect command:

   `docker inspect -f {{.Mounts}} CONTAINER_ID`

## Sharing data between containers

* First, we create a data container that we will later share with another container:

   `docker run -v /data --name data ubuntu:14.04 /bin/bash`

  Then, we create another container to who we mount the data container:

   `docker run -ti --volumes-from data ubuntu:14.04 /bin/bash`

## Copying data to and from containers

* First, we create a test container:

   `docker run -d --name testcopy ubuntu:14.04 sleep 360`

  Then, inside the container, we create a file:

```
   docker exec -ti testcopy /bin/bash`
   cd /root
   echo 'I am in the container' > file.txt
   exit
```

  Now, we retrieve the file with the docker cp command:

   `docker cp testcopy:/root/file.txt .`

  Then, we update the file's content, and we copy it back on the conatainer:

```
   echo "I am now in the host" >> file.txt
   docker cp file.txt testcopy:/root/file.txt
```