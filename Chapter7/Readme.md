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
## Chapter2: Using Docker to test Apache Mesos and Marathon on Docker

* Files:
  * compose-mesos.yml

* Let's build mesos DC made of 4 containers:
 
  ```
  docker-compose -f /vagrant/compose-mesos.yml
  ```
