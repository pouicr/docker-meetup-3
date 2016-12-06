class: center, middle, inverse
# Docker swarm mode

---
## Swarm?

Docker Swarm is native clustering for Docker. It turns a pool of Docker hosts into a single, virtual Docker host.  
This solution is provided by Docker inc.

Alternatives :
* Kubernetes
* Marathon - Mesos
---

## Swarm vs Swarm mode (>1.12)

In order to run swarm you needed:
* Use an external key/value storage (etcd or consul)
* Install docker on each node
* Run specific swarm container

swarm mode:
* Install docker on each node

---

## Concepts

* swarmkit: cluster management and orchestration

* node: member of a cluster (manager or worker)

* task: unit of work (a container definition : image, cmd ...)

* service: a set of replica (task) based on specified image

* load balancing: using ingress load balancing and ipvs

---

## Create your Docker Swarm cluster

1. Initialize
2. Join

---

## Initialization

First, choose a host for the manager role and initialize the cluster:
```bash
docker swarm init --advertise-addr eth1
```

The swarm cluster listen on interface eth1 and display the join token.

To display another time the token :
```bash
docker swarm join-token worker
```

---

## Join

On each members of the cluster :
```bash
docker swarm join  --token $TOKEN 192.168.56.10:2377
```
(using master ip)

---

## Manager and Worker

Two roles are available in swarm:

* manager
 * creation/update/delete service
 * containers orchestration (task scheduling)
 * cluster management (node lifecycle)
 * internal cluster election (single leader)

* worker
 * receive and execute tasks
 * by default manager are also worker
---
class: center, middle, inverse

##  Managers and Workers


![Docker swarm mode](./img/swarm-diagram.png)


---

## Service

A service is the definition of the tasks to execute on the worker nodes.

There are two modes:
* replicated: distribute a specific number of replicas taks among workers
* global: run one tasks on each worker

```bash
docker service create -p 8080:80 --name monservice  --env MYVAR=foo awl-httpd

```
(to be run on a manager)

---

## Service command

The service command (pretty):

```bash
$ docker service inspect --pretty frontend
ID:		c8wgl7q4ndfd52ni6qftkvnnp
Name:		frontend
Labels:
 - org.example.projectname=demo-app
Mode:		REPLICATED
 Replicas:		5
...
ContainerSpec:
 Image:		nginx:alpine
Resources:
Ports:
 Name =
 Protocol = tcp
 TargetPort = 443
 PublishedPort = 4443
```

---

## Service command

```bash
$ docker service ls
ID            NAME      REPLICAS  IMAGE         COMMAND
c8wgl7q4ndfd  frontend  5/5       nginx:alpine
dmu1ept4cxcf  redis     3/3       redis:3.0.6
```

---

## docker ps and service ps

A service is a set of standard containers:

```bash
$ docker service ps redis
ID                 NAME      SERVICE IMAGE    LAST STATE    DESIRED STATE  NODE
wf1x5vqi8lgzlgnpq  redis.1   redis   redis:3  Running 8 s   Running        manager1
ex0d57cqcwoe3jthu  redis.2   redis   redis:3  Running 9 s   Running        worker2
daqg37s9pwayjecrf  redis.3   redis   redis:3  Running 9 s   Running        worker1
olmclyihzx67zsssj  redis.4   redis   redis:3  Running 9 s   Running        worker1
```

---

## Update and Scale

There are command for update and scale service

```bash
docker service update --limit-cpu 2 redis
```

```bash
docker service scale redis=10
```

---

## Advanced cases

* Update Strategy :
 * --update-delay duration          Delay between updates
 * --update-failure-action string   Action on update failure (pause|continue) (default "pause")
 * --update-parallelism uint        Maximum number of tasks updated simultaneously (0 to update all at once) (default 1)


* You can use contrainst for a specified service :
 * node.id 	node's ID 	node.id == 2ivku8v2gvtg4
 * node.hostname 	node's hostname 	node.hostname != node-2
 * node.role 	node's manager or worker role 	node.role == manager
 * node.labels 	node's labels added by cluster admins 	node.labels.security == high
 * engine.labels 	Docker Engine's labels 	engine.labels.operatingsystem == ubuntu 14.04
