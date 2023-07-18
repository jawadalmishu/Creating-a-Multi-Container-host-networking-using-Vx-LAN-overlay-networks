
# Creating a Multi-Container host networking using Vx-LAN overlay networks.




## Introduction
In this tutorial, we will delve into VxLAN technology, Docker containers, and the process of creating and establishing communication between them.

We will have two host machine(ubuntu) and two container in the two host machine. We will ping from one container in first host machine to another container in second host machine.

Please go to this [medium blog](https://medium.com/@mishu667/creating-a-multi-container-host-networking-using-vx-lan-overlay-networks-a3489c3c1567) to get the full article.
## What is VxLAN?
VXLAN (Virtual Extensible LAN) is a network virtualization technology that enables the creation of virtualized Layer 2 networks over existing Layer 3 infrastructure. It allows for the encapsulation of Layer 2 Ethernet frames within Layer 3 UDP (User Datagram Protocol) packets, facilitating the extension of virtual networks across different physical networks or data centers. VXLAN helps overcome scalability limitations in traditional Layer 2 networks by providing a larger address space and allowing for efficient network overlays, making it well-suited for virtualized and cloud environments.
## What is VNI?
VNI stands for Virtual Network Identifier. It is a key component of VXLAN technology. Each VXLAN network is assigned a unique VNI, which serves as an identifier for the virtual network. The VNI acts as a segmentation mechanism, allowing multiple virtual networks to coexist over a shared physical network infrastructure. When encapsulating Layer 2 Ethernet frames into VXLAN packets, the VNI is included in the packet header to distinguish between different virtual networks. This enables the isolation and segmentation of traffic across VXLAN networks, providing the ability to create and manage multiple logical networks on top of a common physical infrastructure.
## What is VTEP?
VTEP stands for VXLAN Tunnel Endpoint. It is a network device or component that serves as the entry and exit point for VXLAN traffic. The VTEP is responsible for encapsulating and decapsulating Layer 2 Ethernet frames into VXLAN packets as they traverse between the virtual and physical networks.
## What is underlay network?
The underlay network refers to the physical network infrastructure that provides connectivity between network devices, such as switches, routers, and servers. It serves as the foundation on which virtual networks and overlays, like VXLAN, are built.

The underlay network typically consists of physical cables, switches, routers, and other networking devices that establish connectivity between different endpoints. It handles the transport of data packets, routing of traffic, and provides the necessary bandwidth and reliability for network communication.

In the context of VXLAN, the underlay network carries the encapsulated VXLAN packets between VTEPs (VXLAN Tunnel Endpoints) over IP (Internet Protocol) or other transport protocols. The underlay network does not have direct knowledge of the virtual networks or overlays running on top of it.

The underlay network’s primary role is to provide efficient and reliable transport of traffic between network devices, while the virtual networks and overlays, such as VXLAN, operate at a higher level to enable network virtualization, segmentation, and isolation.

In summary, the underlay network forms the physical infrastructure that supports the operation of virtual networks and overlays, providing the necessary connectivity and transport for network traffic.
## What is overlay network?
An overlay network is a computer network that is built on top of an existing network infrastructure, such as the internet or a private network. It provides additional functionality or services by encapsulating and tunneling network traffic over the underlying network.

In an overlay network, nodes (computers or network devices) establish virtual connections with each other, forming a logical network on top of the physical network. These virtual connections are typically created using software-defined networking (SDN) techniques or protocols.
## What is Container?
A container is a lightweight, standalone, and executable software package that contains everything needed to run an application, including the code, runtime, system tools, libraries, and configurations. It is based on the concept of containerization, which allows applications to be isolated and run in a consistent environment across different computing environments.

Containers provide a level of abstraction, similar to virtual machines, but with lower overhead and greater efficiency. Unlike virtual machines, which run a complete operating system on top of a hypervisor, containers share the host machine’s operating system kernel. This shared kernel makes containers lightweight, quick to start, and allows for greater resource utilization.
## What is docker?
Docker is an open-source platform that allows you to automate the deployment, scaling, and management of applications using containerization. It provides a way to package software and its dependencies into standardized units called containers.
## Prerequisites:
- Basic familiarity with Linux and networking concepts.
- Vagrant, Git and VirtualBox installed on your machine.
## Step 1: Setting Up the Environment:
First you need to install oracle virtual box, git and vagrant. Now create a new folder name vagrant-vms(you can give any name as your wish). Now go to vagrant-vms folder and create two folder Host 1 and Host 2(you can give any name as your wish).

I make this two folder because i will create two ubuntu virtual machine in to this two folder and also install docker container into this vm(virtual machine) and communicate between them via VxLAN Tunnel. Yes this is the ultimate goal we will achieve today in this tutorial. So, read carefully, i will describe step by step process.

We will need two git bash terminal. Now go to Host 1 folder and open git bash terminal here.Now run this command in the git bash.

```ruby
vagrant init ubuntu/bionic64
```

It will create the ubuntu 18 virtual os in Host 1 folder, but it will take some time.

Now go to Host 2 folder and open git bash terminal here.Now run this command again.

```ruby
vagrant init ubuntu/bionic64
```

It will create the ubuntu 18 virtual os in Host 2 folder.

Now open the oracle virtual box, you will see this two os.

So, succcessfully we have created the virtual machines.

Now, for this project we will give static ip address to this two virtual machine Host 1 and Host 2 manually and also we will make sure that two virtual machine should be in the same network. We will give 192.168.33.10 ip address to Host 1 and 192.168.33.11 ip address to Host 2, so that they are in the same network. For this, please type the comman in the two git bash terminal two power off the vm.

```ruby
vagrant halt
```

Now go to Host 1 folder and you will see the vagrant file there. Open it with Notepad++(any text editor you wish). This script is written in ruby, don’t worry about this script. Go to line number 35. if you don’t see ip address after private network, please add this ip adress.

now remove the ‘#’ sign. It will uncomment this line and will assign ip adress to this vm. Now save this file. Same, you have to go Host 2 folder and uncomment the line number 35 and add ip address 192.168.33.11 and save it.

- Now, again open one git bash for Host 1(you have to go to Host 1 folder and open git bash here) and another git bash for Host 2.
Type this command in the two git bash

```ruby
vagrant up
```

So, far we have two host running with the ip address 192.168.33.10 and 192.168.33.11, check in oracle virtual box. Now check this fighure below, here enp0s8 is the interface of the virtual machine. We have attached our ip addresses to this interface. You should remember this interface enp0s8, because we will need this in later.

- Once the VM is up and running, access the command prompt within the VM by running:

```ruby
vagrant ssh
```

- Now swith to root user. Type inboth git bash.

```ruby
sudo -i
```

Now, just for testing prupose type this command from Host 1 gitbash.

```ruby
ping 192.168.33.11 -c 5
```

You will get output like this.

See, you have successfully ping the Host 2(192.168.33.11). we received 5 packets result as we give in the command. You can also check from host 2 gitbash(ping 192.168.33.10 -c 5).

So, Host 1 and Host 2 can communicate with each other, but our goal is different. We will create docker container inside two host and will ping form each other to connect them.
## Step 2: Docker client installing:

Install docker client and create separate subnet using docker network utility. Now, type this command one by one in git bash.

For Host 1

```ruby
# update the repository and install docker
sudo apt update
sudo apt install -y docker.io

# create a separate docker bridge network 
sudo docker network create --subnet 172.18.0.0/16 vxlan-net

# gitbash output
4951a7fcd71032d8a5b2617eb7c04046b917bf57ef55c42fec52fc77556d3ff3(you may 
have different id)

# list all networks in docker
sudo docker network ls

# gitbash output
NETWORK ID     NAME        DRIVER    SCOPE
982c9e8e9e40   bridge      bridge    local
2c2b3714bca9   host        host      local
223b98ebd6ef   none        null      local
4951a7fcd710   vxlan-net   bridge    local (This is the bridge and it's id)
```

For Host 2:

```ruby
# update the repository and install docker
sudo apt update
sudo apt install -y docker.io

# create a separate docker bridge network
sudo docker network create --subnet 172.18.0.0/16 vxlan-net

# gitbash output
bdc780133e0ee0fcce1599c10bd6699ec49d251cd29e1001f703cad124a3e73b(you may have 
different id)

# list all networks in docker
sudo docker network ls

# gitbash output
NETWORK ID     NAME        DRIVER    SCOPE
6e1b56c794c3   bridge      bridge    local
d1f9b80443c4   host        host      local
922f1ff12422   none        null      local
bdc780133e0e  vxlan-net   bridge    local (This is the bridge and it's id)
```

Here we have created a docker bridge and given a static ip 172.18.0.0(under same network) for host 1 and host 2 and named this bridge vxlan-net in both host. For this, host 1 docker bridge and host 2 docker bridge will take ip address 172.18.0.1.

Now type the command.

For Host 1:

```ruby
# Check interfaces
ip a

# gitbash output
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:3b:7b:b7:3b:2d brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 76043sec preferred_lft 76043sec
    inet6 fe80::3b:7bff:feb7:3b2d/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:2d:bd:e7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.33.10/24 brd 192.168.33.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe2d:bde7/64 scope link
       valid_lft forever preferred_lft forever
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:45:18:6a:d5 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
5: br-4951a7fcd710: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:b1:99:e0:de brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global br-abea36888fcf
       valid_lft forever preferred_lft forever
```

For Host 2:

```ruby
# Check interfaces
ip a

# gitbash output
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:3b:7b:b7:3b:2d brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 71149sec preferred_lft 71149sec
    inet6 fe80::3b:7bff:feb7:3b2d/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:17:9e:b8 brd ff:ff:ff:ff:ff:ff
    inet 192.168.33.11/24 brd 192.168.33.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe17:9eb8/64 scope link
       valid_lft forever preferred_lft forever
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:6a:3a:d8:38 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
5: br-bdc780133e0e: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:39:a5:1b:21 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global br-6d18c8362259
       valid_lft forever preferred_lft forever
```

Now look at the output serial number 5. It’s the docker bridge(For host 1 it’s br-4951a7fcd710 and for host 2 it’s br-bdc780133e0e) and it’s ip address is 172.18.0.1. and if you carefully look up the serial number 3 it’s showing the interface(enp0s8) and it’s ip address(for host 1 it is 192.168.33.10 and for host 2 it is 192.168.33.11). I have told you about interface enp0s8 previously.
## Step 3: Run Docker Container:

Run docker container on top of newly created docker bridge network and try to ping docker bridge.

For Host 1:


```ruby
# running ubuntu container with "sleep 3000" and a static ip
sudo docker run -d --net vxlan-net --ip 172.18.0.11 ubuntu sleep 3000

# gitbash output
07ccc2ed8b1da70a41325eddbeb16f8c67b51916a5faa11f43d6e0c3f67e899c

# check the container running or not
sudo docker ps

# gitbash output
CONTAINER ID   IMAGE     COMMAND        CREATED         STATUS         PORTS     NAMES
07ccc2ed8b1d   ubuntu    "sleep 3000"   7 seconds ago   Up 6 seconds             jolly_khayyam

# check the IPAddress to make sure that the ip assigned properly
sudo docker inspect 07ccc2ed8b1d| grep IPAddress (carefully give the container
 id here)

# gitbash output
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "172.18.0.11",

# ping the docker bridge ip to see whether the traffic can pass
ping 172.18.0.1 -c 2

# gitbash output
PING 172.18.0.1 (172.18.0.1) 56(84) bytes of data.
64 bytes from 172.18.0.1: icmp_seq=1 ttl=64 time=0.047 ms
64 bytes from 172.18.0.1: icmp_seq=2 ttl=64 time=0.044 ms

--- 172.18.0.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1010ms
rtt min/avg/max/mdev = 0.044/0.045/0.047/0.001 ms
```


For Host 2:

```ruby
# running ubuntu container with "sleep 3000" and a static ip
sudo docker run -d --net vxlan-net --ip 172.18.0.12 ubuntu sleep 3000

# gitbash output
d2e7ce1ff3ff58727053a9edf5eeed3be06af921e316914aca1671ed85340620

# check the container running or not
sudo docker ps

# gitbash output
CONTAINER ID   IMAGE     COMMAND        CREATED         STATUS         PORTS     NAMES
d2e7ce1ff3ff   ubuntu    "sleep 3000"   9 seconds ago   Up 8 seconds             affectionate_black

# check the IPAddress to make sure that the ip assigned properly
sudo docker inspect d2e7ce1ff3ff| grep IPAddress(carefully give the container
 id here)

# gitbash output
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "172.18.0.12",

# ping the docker bridge ip to see whether the traffic can pass
ping 172.18.0.1 -c 2

# gitbash output
PING 172.18.0.1 (172.18.0.1) 56(84) bytes of data.
64 bytes from 172.18.0.1: icmp_seq=1 ttl=64 time=0.047 ms
64 bytes from 172.18.0.1: icmp_seq=2 ttl=64 time=0.044 ms

--- 172.18.0.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1010ms
rtt min/avg/max/mdev = 0.044/0.045/0.047/0.001 ms
```

Here we have created docker container which can ping docker bridge. And also we have given static ip(172.18.0.11 for host 1 container, 172.18.0.12 for host 2 container) for each docker container.
## Step 4: Access Docker Container:

Now access to the running container and try to ping another hosts running container via IP Address. Though hosts can communicate each other, conatiner communication should fail because there is no tunnel or anything to carry the traffic.

For Host 1:

```ruby
# enter the running container using exec 
sudo docker exec -it 07ccc2ed8b1d bash (carefully give the container id, you 
should have different container id)

# Now we are inside running container
# update the package and install net-tools and ping tools
apt-get update
apt-get install net-tools
apt-get install iputils-ping

# Now ping the another container
ping 172.18.0.12 -c 2

# gitbash output
PING 172.18.0.12 (172.18.0.12) 56(84) bytes of data.
--- 172.18.0.12 ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 2028ms
```

For Host 2:

```ruby
# enter the running container using exec
sudo docker exec -it d2e7ce1ff3ff bash (carefully give the container id, you 
should have different container id)

# Now we are inside running container
# update the package and install net-tools and ping tools
apt-get update
apt-get install net-tools
apt-get install iputils-ping

# Now ping the another container
ping 172.18.0.11 -c 2

# gitbash output
PING 172.18.0.11 (172.18.0.11) 56(84) bytes of data.
--- 172.18.0.11 ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 2028ms
```

Now type exit to came out from container.

## Step 5: Creating VxLAN Tunnel:

It’s time to create a VxLAN tunnel to establish communication between two hosts running containers. Then attch the vxlan to the docker bridge. Make sure the VNI ID is the same for both hosts.

For Host 1:

```ruby
# check the bridges list on the hosts
brctl show

# gitbash output
bridge name     bridge id               STP enabled     interfaces
br-4951a7fcd710       8000.0242b199e0de       no              veth2466d14
docker0         8000.024245186ad5       no

# create a vxlan
# 'vxlan-demo' is the name of the interface, type should be vxlan
# VNI ID is 100
# dstport should be 4789 which a udp standard port for vxlan communication
# 192.168.33.11 is the ip of another host
sudo ip link add vxlan-demo type vxlan id 100 remote 192.168.33.11 dstport 
4789 dev enp0s8  

# check interface list if the vxlan interface created
ip a | grep vxlan

# gitbash output 
9: vxlan-demo: <BROADCAST,MULTICAST> mtu 8951 qdisc noop state DOWN group default qlen 1000

# make the interface up
sudo ip link set vxlan-demo up

# now attach the newly created vxlan interface to the docker bridge we created
sudo brctl addif br-4951a7fcd710 vxlan-demo (carefully give the bridge id)

# check the route to ensure everything is okay. here '172.18.0.0' part is our concern part.
route -n

#git bash output
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.2.2        0.0.0.0         UG    100    0        0 enp0s3
10.0.2.0        0.0.0.0         255.255.255.0   U     0      0        0 enp0s3
10.0.2.2        0.0.0.0         255.255.255.255 UH    100    0        0 enp0s3
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
172.18.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-4951a7fcd710
192.168.33.0    0.0.0.0         255.255.255.0   U     0      0        0 enp0s8
```

For Host 2:

```ruby
# check the bridges list on the hosts
brctl show

# gitbash output
bridge name     bridge id               STP enabled     interfaces
br-bdc780133e0e       8000.024239a51b21       no              veth8787a02
docker0         8000.02426a3ad838       no 

# 192.168.33.10 is the ip of another host
# make sure VNI ID is the same on both hosts, this is important
sudo ip link add vxlan-demo type vxlan id 100 remote 192.168.33.10 
dstport 4789 dev enp0s8

# check interface list if the vxlan interface created
ip a | grep vxlan

# gitbash output
9: vxlan-demo: <BROADCAST,MULTICAST> mtu 8951 qdisc noop state DOWN group default qlen 1000

# make the interface up
sudo ip link set vxlan-demo up
# now attach the newly created vxlan interface to the docker bridge we created
sudo brctl addif br-bdc780133e0e vxlan-demo (carefully give the bridge id)

# check the route to ensure everything is okay. here '172.18.0.0' part is our concern part.
route -n

# gitbash output
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.2.2        0.0.0.0         UG    100    0        0 enp0s3
10.0.2.0        0.0.0.0         255.255.255.0   U     0      0        0 enp0s3
10.0.2.2        0.0.0.0         255.255.255.255 UH    100    0        0 enp0s3
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
172.18.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-bdc780133e0e
192.168.33.0    0.0.0.0         255.255.255.0   U     0      0        0 enp0s8
```


## Step 6: Testing the connectivity:

Now test the connectivity. It should work now. A Vxlan Overlay Network Tunnel has been created.

For Host 1:

```ruby
# enter the running container using exec 
sudo docker exec -it 07ccc2ed8b1d bash (carefully give the container id, you 
should have different container id)

# ping the other container IP
ping 172.18.0.12 -c 2

# gitbash output
PING 172.18.0.12 (172.18.0.12) 56(84) bytes of data.
64 bytes from 172.18.0.12: icmp_seq=1 ttl=64 time=0.601 ms
64 bytes from 172.18.0.12: icmp_seq=2 ttl=64 time=0.601 ms

--- 172.18.0.12 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1018ms
rtt min/avg/max/mdev = 0.601/0.601/0.601/0.000 ms
```

For Host 2:

```ruby
# enter the running container using exec
sudo docker exec -it d2e7ce1ff3ff bash

# ping the other container IP
ping 172.18.0.11 -c 2

# gitbash output
PING 172.18.0.11 (172.18.0.11) 56(84) bytes of data.
64 bytes from 172.18.0.11: icmp_seq=1 ttl=64 time=0.601 ms
64 bytes from 172.18.0.11: icmp_seq=2 ttl=64 time=0.601 ms

--- 172.18.0.11 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1018ms
rtt min/avg/max/mdev = 0.601/0.601/0.601/0.000 ms
```

See the figure, we have got 100% packets transmission from host 1 to host 2.

See the figure, we have got 100% packets transmission from host 2 to host 1.

And finally we have successfully ping from one container in one host to another container from second host.
