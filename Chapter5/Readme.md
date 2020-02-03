# Chapter5: Kubernetes

## 1. Understanding Kubernetes architecture

* A kubernetes cluster contains:
  * kubenetes master services: provides an API, collect the current state of the cluster, assigbs pods to nodes
  * master storage: all persistent state stored in etcd
  * kubelet: agent running on every nodes and reponsible for driving docker, reporting status to master and setting node resources
  * kubernetes-proxy: runs on every ndode, and provide network endpoint to reach an array of pods

## 2. Networking pods for container connectivity

## 3. Creating a multinode cluster with vagrant

### 1. Minikube

* Files:
   * minikube/Vagrantfile

* The vagrant provider is not available  anymore. They replaced it by a tool called Minikube, who's a playground to test Kubernetes. You can start it by running vagrant:

   ```
   cd minikubube
   vagrant up
   vagrant ssh
   ```

We now have a kubernetes node fully up and running:

   ```
   kubectl get nodes
   ```

### 2. Custom solution

* Files:
   * Vagrantfile
   * ansible/

* As the vagrant provider is not avaiable, I have writted an ansible playbook to deploy a multinode cluster. The cluster is composed of a master and a node. The cluster networking is based on weave. You can start it by running the following commands:

   ```
   vagrant up master node1
   vagrant ssh master
   ```
We now have a multinode kubernetes cluster up and running:

   ```
   kubectl get nodes
   ```

## 4. Starting containers on a Kubernetes cluster with pods

* Files:
   * 2048.yml

* We wrote a pod definition in the file 2048.yml, and use the kubectl command to submit it to he Kubernetes API server:

   ```
   sudo kubectl create -f /vagrant/2048.yml
   sudo kubectl get pods
   ```
Once the image downloaded the container will start running, and you can open your browser at the address http://HOST_IP to start playing 2048.

After having finished to play, you can easily delete the pod:

   `sudo kubectl delete 2048`

##  5. Taking advange of labels for querying kubernetes objects

* Files:
   * 2048.yml

* In large k8s cluster, we need a way to easily query and manipulate sets of objects. This is possible by using labels. They are
key/value pairs declared in the metadata section of an object definition.

* In our case, with the 2048.yml file, we can list pods that have a specific label:

   ```
   kubectl get pods --selector="foo=bar"
   ```

* We also can add labels at runtime using the kubectl label command:

   ```
   kubectl label pods 2048 env=production
   ```

## 6. Using a Replication Controller to manage the number of replicas of a pod

* Files:
   * rs2048.yml

* ReplicaSets (who have replaced the Replication Controller) insure to maintain a stable set of replica pods running at any given
time, to guarantee the availibility of pods.

* We can start our ReplicaSet with the following command:

   ```
   kubectl apply -f /vagrant/rs2048.yml
   ```

* We can also list the available Replica Sets:

   ```
   kubectl get rs
   ```

* Now let's describe our rcgame Replica Set:

   ```
   kubectl describe rs/rcgame
   ```

* We also can resize the Replica Set at runtime. Per example, let's increase it to 5 replicas:

   ```
   kubectl scale --replica=5 rs rcgame
   kubectl get all
   ```

## 7. Running multiple containers in a Pod

* Files:
   * wordpress.yml
   * rswordpress.yml

* It's possible to run more than one container in a pod. In our example we are going to run a mysql with a wordpress container, using the Docker linking
mechanism to allow them to communicate.

   ```
   kubectl apply -f /vagrant/wordpress.yml
   kubectl get all
   ```

* Now we can display the logs of the container in our pod:

   ```
   kubectl logs pod/wp wordpress
   kubectl logs pod/wp mysql
   ```

* As we did for the 2048 game, we can also use a ReplicaSet to have a replica pods to ensure the availability of our wordpress app:

   ```
   kubectl delete pod/wp
   kubectl apply -f /vagrant/wordpress.yml
   kubectl get all
   ```
