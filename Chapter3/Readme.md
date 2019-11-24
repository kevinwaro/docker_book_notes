# Chapter3: Docker networking

## 1. Finding the IP address of a container

* to get the IP address of a container, we can use the docker inspect command and the GO template format:

   ```
   docker run -d --name nginx nginx
   docker inspect --format '{{ .NetworkSettings.IPAddress }}' nginx
   ```

## 2. Exposing a container port on the host

* files:
    * Dockerfile

* let's build a container without any port mapping:

   ```
   docker build -t flask -f Dockerfile .
   docker run -d --name foobar flask
   ```

* we can find the IP address and reach the container:

   ```
   docker inspect --format '{{ .NetworkSettings.IPAddress }}' foobar
   curl http://CONTAINER_IP:5000/hi
   ```
* however, the application can't be reached from outside the host. For this we will need port mapping:

   ```
   docker kill foobar
   docker rm foobar
   docker run -d -p 5000 --name foobar flask
   docker ps
   ```

* we also have the docker port command:

  ```
  docker port foobar 5000
  ```

* we can also use port mapping for UDP ports:

   ```
   docker run -d -p 5000/tcp -p 53/udp flask
   docker ps
   ```

* by default, docker can manipulate the iptables rules. You can see it by listing them:

   ```
   iptables -L
   ```

## 3. Linking containers in Docker

* let's build a 3-tier system to illustrate linking:

   ```
   docker run -d --name database -e MYSQL_ROOT_PASSWORD=root mysql
   docker run -d --link database:db --name web runseb/hostname
   docker run -d --link web:application --name lb nginx
   docker ps
   ```
* as a result of linking, we can display all the environment variables set:

   ```
   docker exec -ti web env | grep DB
   docker exec -ti lb env | grep APPLICATION
   ```

* and so is also updated the hosts file to contain infos about the containers for name resolution:

  ```
  docker exec -ti web cat /etc/hosts
  docker exec -ti lb cat /etc/hosts
  ```
* with docker inspect we can retrieve infos about how the containers are linked:

  ```
  docker inspect -f "{{.HostConfig.Links}}" web
  docker inspect -f "{{.HostConfig.Links}}" lb
  ```

## 4. Understanding Docker container networking

* when starting Docker a network bridge (docker0) is created:

   ```
   sudo apt-get install bridge-utils
   sudo brctl show docker0
   sudo brctl showmacs docker0
   ```
* each time a container is created, an IPtables rule is added:

   ```
   docker run -d python:3.4 python3 -m http.server 1234
   docker ps
   sudo iptables -t nat -L
   sudo iptables -S
   ```
## 5. Choosing a container networking namespace

* let's start a container without networking namespaces:

   ```
   docker run -it --rm --net=none ubuntu:14.04 bash
   ip link show
   route
   ```

* now we start a container with the networking namespace of the host by using --net=host :

   ```
   docker run -it --rm --net=host ubuntu:14.04 bash
   ip link show
   ```

* the final option is to use the network stach of another already using container:

   ```
   docker run -it --rm -h cookbook ubuntu:14.04 bash
   ifconfig
   exit
   docker ps
   docker run -it --rm --net=container:CONTAINER_ID  ubuntu:14.04 bash
   ifconfig
   ```

* you will see that the newly started container will share the same hostname as the first container started, and ofc the
  same IP.

## 6. Configuring the Docker Daemon IP Tables and IP Forwarding settings

* let's disable the defautl behaviour of Docker that allow him to manipulate IPtable:

  ```
  sudo systemctl stop docker.service
  sudo systemctl edit docker.service
  ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --iptables=false --ip-forward=false
  sudo iptables -F
  sudo iptables --delete-chain DOCKER-ISOLATION-STAGE-1
  sudo iptables --delete-chain DOCKER-ISOLATION-STAGE-2
  sudo iptables --delete-chain DOCKER
  sudo iptables --delete-chain DOCKER-USER
  echo 0 > /proc/sys/net/ipv4/ip_forward
  sudo systemctl restart docker.service
  ```

* now let's run a container:

  ```
  docker run -it --rm --net=none ubuntu:14.04 bash
  ping -c 2 8.8.8.8
  ```

* now let's re-enabled the networking setup;

  ```
  echo 1 > /proc/sys/net/ipv4/ip_forward
  docker run -it --rm ubuntu:14.04 bash
  sudo iptables -t nat -A POSTROUTING -s 172.17.0.0/16 -j MASQUERADE
  ```

## 7. Using pipework to understand container networking

* files:
    * pipework/Vagrantfile

* we start a container without networking namespace:
  ```
  docker run -it --rm --net=none --name=cookbook ubuntu:14.04 bash
  ```

* in another terminal, we use pipework to create a bridge and assign an IP address to the container:
  ```
  cd /vagrant
  sudo ./pipework br0 cookbook 192.168.1.10/24@192.168.1.254
  ```

## 8. Setting up a custome bridge for Docker

* we turn off the Docker daemon, and delete the docker0 bridge created by default:

   ```
   sudo systemctl stop docker.service
   sudo ip link set docker0 down
   sudo brctl delbr docker0
   sudo brctl addbr cookbook
   sudo ip link set cookbook up
   sudo ip addr add 10.0.0.1/24 dev cookbook
   ```
* now we edit the startup option of Docker:

   ```
   sudo systemctl edit docker.service
   ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock -b cookbook
   sudo systemctl restart docker.service
   ```
* we can now start a container and list the IP address assigned to it:

   ```
   ip addr show
   ```

* Docker also handled the NAT rules:

   ```
   sudo iptables -L -t nat
   ```

## 9. Using OVS with Docker

* on the docker host, we install the vswitch packages:

   ```
   sudo apt-get install openvswitch-switch
   ```

* we create a switch a bring it up:

   ```
   sudo ovs-vsctl add-br ovs-cookbook
   sudo ip addr add 10.0.0.1/24 dev ovs-cookbook
   sudo ip link set ovs-cookbook up
   ```
* start a container without networking stack, and in a different shell use pipework to create an interface:

  ```
  docker run -it --rm --net=none --name foobar ubuntu:14.04 bash
  cd /vagrant/pipework && sudo su
  ./pipework ovs-cookbook foobar 10.0.0.10/24@10.0.0.1
  ovs-vsctl list-ports ovs-cookbook
  ifconfig
  ```
## 10. Building a GRE tunnel betwenn Docker hosts

* Files:
    * gre/Vagrantfile

* first, we stop Docker and remove this docker0 bridge:

   ```
   sudo su -
   systemctl stop docker.service
   ip link set docker0 down
   ip link del docker0
   ```
* now we can create the GRE tunnel, on the first host:

   ```
   sudo su -
   ip tunnel add foo mode gre local 192.168.33.11 remote 192.168.33.12
   ip addr add 172.17.127.254 dev foo
   ip link set dev foo up
   ip route add 172.17.128.0/17 dev foo
   ```
* on the second host:

   ```
   sudo su -
   ip tunnel add bar mode gre local 192.168.33.12 remote 192.168.33.11
   ip addr add 172.17.255.254 dev bar
   ip link set dev bar up
   ip route add 172.17.0.0/17 dev bar
   ```
* now we edit the docker daemon options, on the first host:

   ```
   systemctl edit docker.service
   ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --bip=172.17.0.1/17 --fixed-cidr=172.17.0.0/17
   systemctl daemon-reload
   systemctl start docker.service
   iptables -A FORWARD -i foo -o docker0 -j ACCEPT
   iptables -A FORWARD -i foo -o docker0 -m state --state ESTABLISHED,RELATED -j ACCEPT
   ```
* on the second host:

   ```
   systemctl edit docker.service
   ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --bip=172.17.128.1/17 --fixed-cidr=172.17.128.0/17
   systemctl daemon-reload
   systemctl start docker.service
   iptables -A FORWARD -i bar -o docker0 -j ACCEPT
   iptables -A FORWARD -i bar -o docker0 -m state --state ESTABLISHED,RELATED -j ACCEPT
   ```

* now each docker host can ping each other:
 
   ```
   ping -c 5 172.17.0.1
   ping -c 5 172.17.128.1
   ```

* but also each container !! 
