# Docker 1.12 swarm concepts
Official Docker Blog read [here] (https://blog.docker.com/2016/07/docker-built-in-orchestration-ready-for-production-docker-1-12-goes-ga/)  

![Docker Swarm](https://github.com/machzqcq/docker-orchestration/blob/master/images/docker-swarm.png)   

If you already have docker hosts (at least 3 hosts) set up , then just get the engine from [here](https://github.com/docker/docker/releases). If NOT, continue reading.

# How to setup

- Install Vagrant 
- git clone this repo
- cd <repo>
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

- `vagrant@smgr:~$ docker swarm init --auto-accept worker,manager --listen-addr 192.168.33.10:2377` - copy the <random_secret>
- `vagrant@snode1:~$ docker swarm join --secret <random_secret> --manager --listen-addr 192.168.33.20:2377 192.168.33.10:2377` - Join a manager
- `vagrant@snode2:~$ docker swarm join --secret <random_secret> --listen-addr 192.168.33.30:2377 192.168.33.10:2377` - Join as worker  

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

# Multiple Services 
Assuming the swarm is up and running with 2 managers and 1 worker node as above  

### example voting app
`git clone https://github.com/machzqcq/example-voting-app.git` See the micro-service app architecture on the README for the repo  

## Build and start db service
`docker service create --replicas 1 --name db -p 5432:5432/tcp postgres:9.4` - Verify `netstat -aln | grep 5432` Should get back the postgres service as listening  

## Build and start redis service
`docker service create --name redis -p 6379:6379/tcp redis:3.2.1-alpine` - Verify `netstat -aln | grep 6379`  

`docker service ls` list all services and their status  
`docker service tasks redis` list all tasks for redis i.e. the # of replicas created to serve redis service  

## Build and start vote service
Step into vote folder and execute `docker build --no-cache -t vote -f Dockerfile.`  

`docker service create --replicas 3 --name vote -p 5000:80 vote:latest python app.py` This will create 3 tasks for the service vote and distributes it across the swarm. There is currently a bug with 1.12.0-rc4, where if the vote image is not available on swarm nodes, then the containers don't start (Workaround is to build the images on all swarm nodes)  

Hit http://192.168.33.10:5000 and you can see the containers at the bottom keep changing (though round-robin is expected, currently it doesn't seem so)  

## Build result service
Step into results folder and execute `docker build --no-cache -t result -f Dockerfile .`  
`docker service create --replicas 2 --name result -p 5001:80/tcp result:latest nodemon --debug server.js` This will create 2 tasks for the service result and distributes across the swarm  

Hit http://192.168.33.10:5001 and you can see the container ids at the bottom of the screen keep changing  

# Visualize

I found cAdvisor from Google and weave Scope to be good enough for a developer's needs. Containers launched on multiple hosts however require a bit more networking solutions like weavenet+weavescope, where weavenet is commercial. That said, the relatively open and free ones are also there to get us started.  

```
sudo curl -L git.io/scope -o /usr/local/bin/scope
sudo chmod a+x /usr/local/bin/scope
scope launch
```  





