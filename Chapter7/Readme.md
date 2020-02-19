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
  docker swarm join --token <TOKEN> 172.16.1.11:2377
  ```

* We can now check of what is made of the swarm cluster:

  ```
  vagrant ssh swarmhead
  docker node ls
  ```

## Chapter8: Orchestrating Containers with Ansible Docker module

* Files:
  * ansible/Vagrantfile
  * ansible/ansible.cfg
  * ansible/inventory
  * ansible/playbooks

* We can use Ansible to manage and orchestrate Docker:

  ```
  ansible-playbook -i inventory playbooks/wordpress.yml -u vagrant
  ansible-playbook -i inventory playbooks/dock.yml -u vagrant
  ```

## Chapter9: Using Rancher to manage Containers on a Cluster of Docker Hosts

* Files:
  * rancher/Vagrantfile

* Rancher is a solution who allows you to manage and orchestrate your Kubernetes Cluster without to have to deal with complex
YAML files. Everything is done through an UI.

* To start playing with it, let's bring up our vms:

  ```
  vagrant up master node1 node2
  ```
* Then, open your browser and go at the address: https://172.16.1.11. It will ask you to create a password, and another few questions.

* After that, you will have access to the Rancher Dashboard. Start playing with it and create a new cluster. A command will be given to you to add the 2 other nodes to the cluster. Paste the command and run them on each of the nodes

   ```
   vagrant ssh node1
   sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.3.5 --server https://172.16.1.11 --token <TOKEN> --ca-checksum <CHECKSUM> --worker
   vagrant ssh node2
   sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.3.5 --server https://172.16.1.11 --token <TOKEN> --ca-checksum <CHECKSUM> --worker
   ```

* You now have a fully configured Kubernetes cluster.
   
