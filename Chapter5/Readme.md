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
