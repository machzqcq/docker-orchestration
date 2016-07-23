# Docker 1.12 swarm concepts
Using this setup to teach myself docker 1.12 swarm  

Get latest docker engine 1.12 from [here](https://github.com/docker/docker/releases).

# How to setup

- Install Vagrant 
- git clone this repo
- vagrant up  

From terminal, type `vagrant status`. This should show 3 nodes - smgr, snode1, snode2. The 's' stands for swarm  

# Get Docker

- `vagrant ssh smgr` will put you inside smgr node shell
- `curl -fsSL https://test.docker.com/ | sh` will install latest docker (will update docker if already present and installed in this manner). 
- `docker version` should give you the version installed  

Example:  

```
vagrant@smgr:~$ sudo usermod -aG docker vagrant
vagrant@smgr:~$ docker version
Client:
 Version:      1.12.0-rc4
 API version:  1.24
 Go version:   go1.6.2
 Git commit:   e4a0dbc
 Built:        Wed Jul 13 03:54:54 2016
 OS/Arch:      linux/amd64

Server:
 Version:      1.12.0-rc4
 API version:  1.24
 Go version:   go1.6.2
 Git commit:   e4a0dbc
 Built:        Wed Jul 13 03:54:54 2016
 OS/Arch:      linux/amd64
```  

# Swarm Creation

- `vagrant@smgr:~$ docker swarm init --auto-accept worker,manager --listen-addr 192.168.33.10:2377` - copy the random secret
- `vagrant@snode1:~$ docker swarm join --secret 00u92trlm9u9hpcwxpcu7f9km --manager --listen-addr 192.168.33.20:2377 192.168.33.10:2377` - Join a manager
- `vagrant@snode2:~$ docker swarm join --secret 00u92trlm9u9hpcwxpcu7f9km --listen-addr 192.168.33.30:2377 192.168.33.10:2377` - Join as worker  

This will create a 3 node swarm cluster, with 2 managers (one being leader) and one worker.  
`docker swarm --help` Practice other sub commands for swarm for eg. inspect  

### Leave swarm
`docker swarm leave --force` executed on a node that is a manager will cause it to leave.  
`docker swarm leave` leave the swarm (on a worker node)


# Swarm Nodes

`docker node ls` lists the node and their current roles. Recommended generally to have a 3 node swarm with each node being a manager and one manager being the leader. If the role column is empty, then that node is a worker  

`docker node promote snode1` will promote snode1 to a manager  
`docker node demote snode1` will demote snode1 to a worker  

# Services 
One any manager node in the swarm   

`docker service create --replicas 1 --name hub -p 4444:4444/tcp selenium/hub:latest` deploy the  selenium hub on the swarm as a service. That means accessing http://192.168.33.10:4444/grid/console OR http://192.168.33.20:4444/grid/console OR http://192.168.33.30:4444/grid/console should all return the UI. The way to think about it is , now all these nodes are tied together into a swarm i.e. one single end point machine.  

`docker service tasks hub` lists the instances of the service i.e. tasks information, including the nodes on which the instances of the service are running. In the above example, check by doing `netstat -aln | grep 4444` on each of the nodes. Though there is only one container, the routing mesh for the service ensures that the service is listening on 4444 on all nodes, which makes the swarm one single entity  

`docker service scale hub=4` will launch 4 containers on any of the nodes in the swarm. Typically goes in round-robin starting with the workers. Can be scaled down as well by specifying the number.  

### Failover
Test the failover by doing `vagrant halt snode1` for e.g. of course assuming that the service you created above has spawned a container on snode1. As soon as the machine goes down, docker swarm automatically rebalances the cluster and creates new containers and ensures that the serviceability of 4 instances of the service is maintained






