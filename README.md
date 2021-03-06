![Docker Swarm](https://github.com/machzqcq/docker-orchestration/blob/master/images/docker-swarm.png)   

# What can you expect 

- Learn to set up a docker swarm (understand terminology and relate to docker swarm implementation). Let swarm manage TLS, certificates, load balancing and key rotation etc.
- Specify DESIRED STATE of your COMPOSITE APPLICATION and let SWARM handle the complexities of bringing it up for you (of course as long as you have dockerized your application following best practices)
- We use Virtualbox and Vagrant to simulate swarm and deploy application on top of it , however the examples below have been tested on AWS EC2 cluster service, docker data center etc.
- Take a composite-service(including micro-services) application and swarm it (aka. deploy your application to docker swarm)
- Test failover
- Test rolling deployments (upgrade components of your application without without blackout windows i.e. zeror downtime to customers accessing your application even when updating apps)
- Take [example-voting-app](https://github.com/machzqcq/example-voting-app.git), a micro-service application with UI, api, worker, db micro-services and swarm it
- Application Monitoring and Visualization -  I found it very helpful to run [cAdvisor](https://github.com/google/cadvisor) or [weaveScope](https://github.com/weaveworks/scope) and visualize the swarm, containers, hosts as they come up and go down

# Assumptions

- Know fundamental docker concepts - docker client, api and engine commands and sub-commands
- Know how to operate and manage virtualbox images (using vagrant)
- Intermediate *nix knowledge - ssh, navigate, start stop services, file i/o etc.
- yaml format (and the sensitivity to spaces)
- Done at least some ops/sysadmin work in the past and knows fundamentals of distributed systems (horizontal scaling, application deployments on distributed systems etc. )
- Willingness to try and NOT give up on initial failures (aka. look up online and fix simple glitches that might be specific to your network, hardware, software setup)

## Key Points for Swarm in 1.12

- No external data store required (previous to docker 1.12, you had to set up a KV store externally and go through hoops)
- Secure by default (* barring some issues) - i.e. TLS setup automatically and all certs are managed/refreshed internally
- Automated Key rotation
- Rolling updates (for e.g. new versions of api's can be deployed without brining the entire system down etc.)
- Health Checks
- Container-native load balancing  

Btw - batteries included (at the heart of docker always)


## CLUSTER TOPOLOGY


### Node (manager and worker)
Fudamental unit of a swarm. 2 types - manager & workers. Swarm is created by churning multiple nodes. Node is any machine that runs Docker 1.12  

### Communication Protocol
Manager nodes communicate in a protocol called "raft", allowing them exchange information with strong consistency. Workers communicate with "gossip", allows them to share information bulk. Though it is eventual consistent, the information is shared very fast and allows workers to scale massively.  

Managers and workers communicate using another protocol called [gRPC](http://www.grpc.io/docs/). It is build on http2, so it allows to communicate with internet very easily, so it works through proxy's etc. Also this protocol is versioned, so different versions of workers can talk to managers easily. Typical cluster has 3,5 or 7 max. manager.

### Role
The role of a node is not static, we can promote or demote as needed.

### KV Store in Quorum
KV Store is embedded within, no need to setup a separate one, so no dependency on external infra. A side effect is that they communicate with embedded TLS, so no separate security infra is needed. The embedded store massively improves performance. Every single manager has the entire desired state cached in memory, enabling them to make fast decisions.  


### Security &  TLS
Every node in the swarm is identified by a Cryptographic Identity that is signed by a central authority and centrally managed. There is no way that one node can assume identify of other node without us noticing.

- Every manager has CA built in.
- A new node needs a join token to join the swarm. It contains the fingerprint of swarm CA and some encoded information. So the worker can trust that it is joining the right entity. The token contains a secret , so that the manager can trust the worker is currently joining is authorized to join the swarm.
- The worker will generate a cert and give it to manager. The manager will sign it and give it back to the worker and from there on the trust relationship is established.
- Appro. every 3 months, the certs expire. Few weeeks before it expires, the worker generates new certs and then send it to manager for signing, once it gets back, swaps out the old one with new one


## ORCHESTRATION

![Swarm Orchestration](https://github.com/machzqcq/docker-orchestration/blob/master/images/Swarm-orchestration-flow.png)  

- User declares the DESIRED STATE of application
- Swarm API accepts the definition and stores the state 
- The orchestrator reconciles between the DESIRED STATE and ACTUAL STATE of the application
- Orchestrator passes the reconciliation plan to scheduler to go implement the DESIRED STATE
- Scheduler assigns the implementation plan to Dispatcher
- Dispatcher passes the tasks to the worker nodes (managers can be workers too) in the swarm, that rebalances the swarm to reach DESIRED STATE

## Quick Start (using docker-machine)
There is a more elaborate set up after this section, however you can quickly get started as below.

- `for N in 1 2 3 4 5; do docker-machine create --driver virtualbox node$N; done` - will start 5 virtual machines that uses busybox image and docker already present as part of the image
- Here is how it looks
```
pradeep@seleniumframework.com$ docker-machine ls
NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER    ERRORS
node1   -        virtualbox   Running   tcp://192.168.99.100:2376           v1.12.2
node2   -        virtualbox   Running   tcp://192.168.99.101:2376           v1.12.2
node3   -        virtualbox   Running   tcp://192.168.99.102:2376           v1.12.2
node4   -        virtualbox   Running   tcp://192.168.99.103:2376           v1.12.2
node5   -        virtualbox   Running   tcp://192.168.99.104:2376           v1.12.2
```  

- `docker-machine ssh node1` - step into one of the nodes
- Initialize swarm
```
docker swarm init --advertise-addr 192.168.99.100 
Swarm initialized: current node (ddec2zwjaes2kpkmnshkmpstq) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-3jv31q64ofkl3j20d8lxeq2eejyhcde7h1o25p6c0i1b1a2e3x-az1nzpiuxaa1ny8te8yocnv3i \
    192.168.99.100:2377
    

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```  

- Exit from node1 and `docker-machine ssh node2`  
- Join node2 as worker
```
docker swarm join \
    --token SWMTKN-1-3jv31q64ofkl3j20d8lxeq2eejyhcde7h1o25p6c0i1b1a2e3x-az1nzpiuxaa1ny8te8yocnv3i \
    192.168.99.100:2377
```  

- Join other nodes
```
for NODE in node3 node4 node5; do
  docker-machine ssh $NODE docker swarm join \
    --token SWMTKN-1-3jv31q64ofkl3j20d8lxeq2eejyhcde7h1o25p6c0i1b1a2e3x-az1nzpiuxaa1ny8te8yocnv3i \
    192.168.99.100:2377
done
```   

- Promote node2 and node 3 - `docker node promote node2 node3 ` - so that we have 3 manager nodes, which is the recommended cluster with fault tolerance

- Type `docker node ls` from node1 and it looks something like this, where one of the 3 managers is master and rest are workers
```
docker@node1:~$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
4pq9ixqwt3j1kn0zjwew009cl    node3     Ready   Active        Reachable
4y1rx2xskffge1eabydvenu8r    node2     Ready   Active        Leader
894ntux8vmz3j9bblg44d4ffx    node5     Ready   Active
ba212hjrtmbtbk96ixxxbjh46    node4     Ready   Active
e0253fetpvgcrqhnsz942uu98 *  node1     Ready   Active        Reachable
```  

- Use swarm visualizer (open source project)
```
docker@node1:~$ docker run -it -d -p 8080:8080 -v /var/run/docker.sock:/var/run/docker.sock manomarks/visualizer
docker@node1:~$ docker service create \
>   --name=viz \
>   --publish=8080:8080/tcp \
>   --constraint=node.role==manager \
>   --mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
>   manomarks/visualizer
2e0f44sqgxybsm6behumsbu2b
```  

- Then access the UI as http://192.168.99.100:8080  

![Swarm Visualizer](https://github.com/machzqcq/docker-orchestration/blob/master/images/swarm-visualizer.png)



# Concepts explained in Video
Watch [this](https://vimeo.com/189520997) &  [this](https://vimeo.com/189543080) video for full demonstration of the below concepts  

- CREATE SERVICE ON SWARM
- LIST TASKS (CONTAINERS/INSTANCES) OF SERVICE
- PUBLISH PORTS
- RECONCILIATION OF DESIRED STATE
- SCALING UP/DOWN
- ROLLING UPDATES/DEPLOYMENTS
- GLOBAL SERVICES
- FAULT TOLERANCE
- SERVICE DISCOVERY & LOAD  BALANCING

# How to setup (USING VAGRANT)

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

- `vagrant@smgr:~$ docker swarm init --auto-accept worker,manager --listen-addr 192.168.33.10:2377` - copy the random_secret
- `vagrant@snode1:~$ docker swarm join --secret <random_secret> --manager --listen-addr 192.168.33.20:2377 192.168.33.10:2377` - Join a manager
- `vagrant@snode2:~$ docker swarm join --secret <random_secret> --listen-addr 192.168.33.30:2377 192.168.33.10:2377` - Join as worker  

This will create a 3 node swarm cluster, with 2 managers (one being leader) and one worker.  
`docker swarm --help` Practice other sub commands for swarm for eg. inspect  

### Leave swarm (not needed, only to practice)
`docker swarm leave --force` executed on a node that is a manager will cause it to leave.  
`docker swarm leave` leave the swarm (on a worker node)


# Swarm Nodes

`docker node ls`   
- lists the node and their current roles. 
- Recommended generally to have a 3 node swarm with each node being a manager and one manager being the leader. 
- If the role column is empty, then that node is a worker   

### Promote and demote nodes

`docker node promote snode1`   
`docker node demote snode1`(optional)

# Services (single service)
On any manager node in the swarm   

### Create Service

`docker service create --replicas 1 --name hub -p 4444:4444/tcp selenium/hub:latest`  
- deploy the  selenium hub on the swarm as a service. 
- That means accessing http://192.168.33.10:4444/grid/console OR http://192.168.33.20:4444/grid/console OR http://192.168.33.30:4444/grid/console should all return the UI. 
- The way to think about it is , now all these nodes are tied together into a swarm i.e. one single end point machine.   

### List Service Tasks

`docker service tasks hub`  
- lists the instances of the service i.e. tasks information, including the nodes on which the instances of the service are running. 
- In the above example, check by doing `netstat -aln | grep 4444` on each of the nodes. 
- Though there is only one container, the routing mesh for the service ensures that the service is listening on 4444 on all nodes, which makes the swarm one single entity   

### Scale service up

`docker service scale hub=4`   
- will launch 4 containers on any of the nodes in the swarm. 
- Typically goes in round-robin starting with the workers. 
- Can be scaled down as well by specifying the number.  

### Failover
- Test the failover by doing `vagrant halt snode1` for e.g. of course assuming that the service you created above has spawned a container on snode1. 
- As soon as the machine goes down, docker swarm automatically rebalances the cluster and creates new containers and ensures that the serviceability of 4 instances of the service is maintained

# Multiple Services 
Assuming the swarm is up and running with 2 managers and 1 worker node as above  

### example voting app (A composite application)
`git clone https://github.com/machzqcq/example-voting-app.git`  
- See the micro-service app architecture on the README for the repo  

## Build and start db service
`docker service create --replicas 1 --name db -p 5432:5432/tcp postgres:9.4`   
- Verify `netstat -aln | grep 5432` - Should get back the postgres service as listening  

## Build and start redis service
`docker service create --name redis -p 6379:6379/tcp redis:3.2.1-alpine`  
- Verify `netstat -aln | grep 6379`  

`docker service ls` list all services and their status  
`docker service tasks redis` list all tasks for redis i.e. the # of replicas created to serve redis service  

## Build and start vote service
- Step into vote folder `cd vote`
- execute `docker build --no-cache -t vote -f Dockerfile.` - will build the vote docker image
- `docker service create --replicas 3 --name vote -p 5000:80 vote:latest python app.py` -  This will create 3 tasks for the service vote and distributes it across the swarm. There is currently a bug with 1.12.0-rc4, where if the vote image is not available on swarm nodes, then the containers don't start (Workaround is to build the images on all swarm nodes)    

## Access Vote app (UI)
- Hit http://192.168.33.10:5000 and you can see the containers at the bottom keep changing  

## Build result service
- Step into results folder - `cd results`
- execute `docker build --no-cache -t result -f Dockerfile .`  
- `docker service create --replicas 2 --name result -p 5001:80/tcp result:latest nodemon --debug server.js` This will create 2 tasks for the service result and distributes across the swarm   

## Access Result app (UI)

- Hit http://192.168.33.10:5001 and you can see the container ids at the bottom of the screen keep changing  

# Monitoring and Visualization

I found cAdvisor from Google and weave Scope to be good enough for a developer's needs. Containers launched on multiple hosts however require a bit more networking solutions like weavenet+weavescope, where weavenet is commercial. That said, the relatively open and free ones are also there to get us started.  

```
sudo curl -L git.io/scope -o /usr/local/bin/scope
sudo chmod a+x /usr/local/bin/scope
scope launch
```    

# Docker Swarm 

Official Docker Blog read [here](https://blog.docker.com/2016/07/docker-built-in-orchestration-ready-for-production-docker-1-12-goes-ga/)
