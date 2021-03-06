# Chapter 2: Image creation and sharing

## 1. Keeping changes made to a container by committing to an image

* first, we create a container, in which we run an apt-get update:

```
   $ docker run -t -i ubuntu:14.04 /bin/bash
   # apt-get update
   # exit
```

* even if the container has exited, it's still available until we remove it (with docker rm). Before that,
let's create a new image called ubuntu:update:

   `docker commit CONTAINER_ID ubuntu:update`

* we can now safely remove the container. The changes made inside the container can be inspected with the docker diff command:

   `docker diff CONTAINER_ID`

## 2. Saving images and containers as Tar files for sharing

* export a container in a tarball:

   `docker export CONTAINER_ID > update.tar`

* import a container from a tarball:

   `docker import - update < update.tar`

* save an image:

   `docker save -o update1.tar update`

* load an image:

   `docker load < update1.tar`

The 2 methods are similar except:
* saving an image will keep its history
* exporting a container will squash the history

## 3. Writing your first Dockerfile

* files:
    * Dockerfile
    * Dockerfile2

* let's build an image from a dockerfile that will allow us to create a container that executes an echo command (see Dockerfile):

   ```
   docker build .
   docker images
   docker run IMAGE_ID Hi Docker !
   ```

* in the dockerfile, instead of ENTRYPOINT, we can use CMD that will force the container to use a predefinite command. The command
passed in CMD can also be overwritten by a command passed in argument:

   ```
   docker build -f ./Dockerfile2 .
   docker images
   docker run IMAGE_ID /bin/date
   ```
* you can also set a tag for your images, instead of to have a randomly generated CONTAINER_ID:

   ```
   docker build -t cookbook:kevin -f ./Dockerfile2
   docker images
   docker run cookbook:kevin /bin/date
   ```
## 4. Packaging a flask app inside a container

* files:
    * hello.py
    * Dockerfile3

* let's build the image from the Dockerfile:

   ```
   docker build -t flask -f ./Dockerfile3 .
   docker images
   docker run -d -P flask
   docker ps
   ```

* then run a curl to  http://192.168.33.10:PORT/hi and you will see that it returns "hello world".

## 5. Optimizing your Dockerfile by following best practices

* files:
    * Dockerfile4

* here are the main ideas about Dockerfile best practices:
    * Run a single process per container.
    * Do not assume your container will live on. They are ephemeral and will be restarted.
    * Use a .dockerignore file to exclude file and directories from being copied during the build process.
    * Use official images from DockerHub.
    * Always minimize the number of layers of the images.

* let's build the image from the Dockerfile:

   ```
   docker build -t flask -f ./Dockerfile4 .
   docker images
   docker run -d -P flask
   docker ps
   ```
* then run a curl to  http://192.168.33.10:PORT/hi and you will see that it returns "hello world".

## 6. Versioning an image with Tags

* let's rename the classic ubuntu image to foobar:

   ```
   docker images
   docker tag ubuntu foobar
   docker tag ubuntu:14.04 foobar
   docker images
   docker tag ubuntu:14.04 foobar:cookbook
   ```
## 10. Using ONBUILD images

* files:
    * Dockerfile5

* we can use the ONBUILD feature to use in a container the directives declared in a parent image.
