# Chapter7: The Docker ecosystem

## Chapter1: Using Docker Compose to create a wordpress site

* Files:
  * docker-compose.yml

* In the recipe 1.16 we created a wordpress site based on a multicontainer setup. Now we will use docker-compose:

   ```
   docker-compose -f /vagrant/docker-compose.yml up -d
   docker-compose ps
   ```
* On the container are created, we can manage them:

   ```
   docker-compose stop wordpress
   docker-compose ps
   docker-compose stop wordpress
   ```
* Finally, we can get rid of them with the following command:

  ```
  docker-compose -f /vagrant/docker-compose.yml rm
  ```

## Chapter2: Using Docker to test Apache Mesos and Marathon on Docker

* Files:
  * compose-mesos.yml

* Let's build mesos DC made of 4 containers:
 
  ```
  docker-compose -f /vagrant/compose-mesos.yml up -d
  ```

* Then open your browser at http://192.168.33.10:5050 and you will have access to the Mesos UI.

## Chapter3: Starting Containers on a cluster with Docker Swarm

* Files:
  * swarm/Vagrantfile

* The Docker swarm mode is a cluster management and orchestration feature embedded in the Docker Engine. A swarm consists of
multiple Docker hosts.

* Let's build a swarm of 3 hosts, made of a manager and 2 workers:

  ```
  vagrant up swarhead
  ```

* At the end of the provisionning, a command to be run on the workers will be displayed. This command will make them join the
swarm:

  ```
  docker swarm join --token <TOKEN> 172.16.1.11:2377
  ```

* Let's start the 2 workers, and provision them. Then we will make them join the swarm:

  ```
  vagrant up swarm1 swarm2
  vagrant ssh swarm1
  docker swarm join --token <TOKEN> 172.16.1.11:2377
  vagrant ssh swarm2
  docker swarm join --token <TOKEN> 172.16.1.11:2377<Paste>
  ```

* We can now check of what is made of the swarm cluster:

  ```
  vagrant ssh swarmhead
  docker node ls
  ```
