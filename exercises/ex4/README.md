# Docker to Helm Hands-On Labs

## Exercise 4: Docker Swarm

**STOP**! Please ensure that you have met all the prereqisites as mentioned [earlier](../../README.md).

### Azure Container Service

Azure Container Service (ACS) provides a way to simplify the creation, configuration, and management of a cluster of virtual machines that are preconfigured to run containerized applications. 

Using an optimized configuration of popular open-source scheduling and orchestration tools, ACS enables deployment and management of container-based applications on Microsoft Azure.

We will look at Docker Swarm as an orchestrator in this exeercise and will subsequently look at Kubernetes in later exercises. **Please note that some of the regions don't support Docker Swarm in Swarm mode yet. They support Docker Swarm in legacy mode. To spin up clusters in Docker Swarm mode, Use Docker CE as the orchestrator**.

### Services with Docker Swarm

##### ssh into the Docker Swarm master

ssh into the master of the Docker swarm cluster created earlier with the following command

```
ssh <ip-of-master>
```

Let's run the following command again to verify that the cluster is in good shape.

```
docker node ls
```

Which should yield an output about the Docker Swarm cluster that looks something like below. It shows the cluster running with a single master and three agents.

```
ID                           HOSTNAME                         STATUS  AVAILABILITY  MANAGER STATUS
8e8qa1vm66534w5hoc0o5v9pt *  swarmm-master-26830735-0         Ready   Pause         Leader
i80wvdub8g47eecem8fqxx8kn    swarmm-agentpool-26830735000000  Ready   Active        
lx5ifv7o16ri19xjb189ig12d    swarmm-agentpool-26830735000004  Ready   Active        
mg7sinoolwjkxmyxc6lc79wal    swarmm-agentpool-26830735000003  Ready   Active
```

#### Deploying a Service

Let's deploy a service to the swarm mode cluster with the following command.

```
docker service create --replicas 1 --name nginx --publish 80:80 nginx
```

This creates a service named `nginx` based on the container image `nginx` from the Docker public repo, forwarding the port 80 of the container to the host.

Let's list the service with the following command.

```
docker service ps nginx
```

Which should yield an output that looks something like below.

```
ID            NAME     IMAGE         NODE                             DESIRED STATE  CURRENT STATE               ERROR  PORTS
s5d50erb5p35  nginx.1  nginx:latest  swarmm-agentpool-26830735000000  Running        Running about a minute ago     
```

Which shows that the orchestrator has scheduled the app on the agent(s) and in this case it is `swarmm-agentpool-26830735000000`. Let's `ssh` to the host running the container using the following command.

```
ssh $(docker service ps nginx | grep nginx | awk '{print $4}')

```

Running the following command on the agent
```
docker ps -a
```

Should yield an output that looks something like below. It shows the image running inside the container orchestarted by Docker Swarm on the cluster.



```
CONTAINER ID        IMAGE                                                                           COMMAND                  CREATED             STATUS              PORTS               NAMES
173a0714590a        nginx@sha256:aa1c5b5f864508ef5ad472c45c8d3b6ba34e5c0fb34aaea24acf4b0cee33187e   "nginx -g 'daemon ..."   9 minutes ago       Up 9 minutes        80/tcp              nginx.1.s5d50erb5p35lc3j4fgos0eae
```

To verify that the app is running, point your browser to `http://<ip-of-master>` or you can also run the following command from a shell on the laptop.

```
curl <ip-of-master>
```

which should yield an output that begins with something like below.

```
Welcome to nginx!
```

Let's stop the service with the following command

```
docker service rm nginx
```

Let's spin up the `spring-boot` service with the following command.


```
docker service create --replicas 1 --name spring-boot --publish 8080:8080 ragsns/spring-boot
```

Verify the service is working via the following command

```
curl <ip-of-master>/8080/env | grep -i hostname
```

Which yields an output that looks something like below

```
HOSTNAME = 5b161c1e8eaf
```

#### Application Self-Healing

Now, we'll leverage the application self-healing property of the framework by explicitly terminating the application with the following command.

```
curl <ip-of-master>/8080/exit
```

which should yield an output that looks something like below indicating the application terminated

```
curl: (52) Empty reply from server
```

Let's try to check the service again with the following command (**Give it a few seconds**)

```
curl <ip-of-master>/8080/env | grep -i hostname
```

The output this time will look a bit different as below.

```
HOSTNAME = 9a0e8506e8e6
```

What happened here? To understand that, let's try the following command.

```
docker service ps spring-boot
```

which should yield an output that looks something like below.

```
ID            NAME               IMAGE                      NODE                             DESIRED STATE  CURRENT STATE           ERROR  PORTS
8t4cdif6gfwu  spring-boot.1      ragsns/spring-boot:latest  swarmm-agentpool-26830735000000  Running        Running 3 minutes ago          
oj3exne6eo5j   \_ spring-boot.1  ragsns/spring-boot:latest  swarmm-agentpool-26830735000000  Shutdown       Complete 3 minutes ago         
```

Essentially the framework monitored the running application and when it exited, it spawned the image in a different container maintaining the replicas as 1 (which is the desired state).

#### Scaling a Service

Now lets, scale the application by creating 2 replicas by running a command as below.

```
docker service scale spring-boot=2
```

which should yield an output confirming the scaling.

You can `inspect` the app bu running the following command

```
docker service inspect --pretty spring-boot
```

Which should yield an output that looks something like below indicating that the application was scaled horizontally running 2 instances.


```
ID:		r4s0zuh99ixd32bumda70d14n
Name:		spring-boot
Service Mode:	Replicated
 Replicas:	2
Placement:
UpdateConfig:
 Parallelism:	1
 On failure:	pause
 Max failure ratio: 0
ContainerSpec:
 Image:		ragsns/spring-boot:latest@sha256:c069009657927d95dec371b4e59945cacebea73d5918161097237e28048f18e8
Resources:
Endpoint Mode:	vip
Ports:
 PublishedPort 8080
  Protocol = tcp
  TargetPort = 8080
``` 

Now, lets the run the following command about 3-4 times

```
curl <ip-of-master>/8080/env | grep -i hostname
```

The output will look something like below

```
HOSTNAME = 5edd66fba7be
HOSTNAME = 9a0e8506e8e6
HOSTNAME = 5edd66fba7be
```

Notice that it gives a clue of the running application which has 2 instances and the traffic is load balanced between the 2 instances.

Now, run the command

```
docker service ps spring-boot | grep -i running
```

The output below indicates the 2 instances running on 2 seperate agents.

```
8t4cdif6gfwu  spring-boot.1      ragsns/spring-boot:latest  swarmm-agentpool-26830735000000  Running        Running 18 minutes ago          
8t4n1a1feeef  spring-boot.2      ragsns/spring-boot:latest  swarmm-agentpool-26830735000004  Running        Running 8 minutes ago 
```

Let's scale the service to 10 replicas.

```
docker service scale spring-boot=10
```

#### Performing rolling upgrades

You can update to service via rolling upgrades so that some of them are serving the old app and some of them are serving the new app in such a way that the users are least impacted.

Let's update `spring-boot` to the `v2` version via a rolling upgrade.

```
docker service update --image ragsns/spring-boot:v2 spring-boot 
```

Now, let's run the following command

```
docker service ps spring-boot

```

The command yields an output that looks something like below, which shows that rolling upgrades of the application is propgating across the cluster. Notice that some of the instances are running the `v2` version whereas some others are running the older version.


```
ID            NAME               IMAGE                      NODE                             DESIRED STATE  CURRENT STATE               ERROR  PORTS
pjm744w2tmns  spring-boot.1      ragsns/spring-boot:v2      swarmm-agentpool-26830735000000  Ready          Preparing 9 seconds ago            
8t4cdif6gfwu   \_ spring-boot.1  ragsns/spring-boot:latest  swarmm-agentpool-26830735000000  Shutdown       Running 9 seconds ago              
oj3exne6eo5j   \_ spring-boot.1  ragsns/spring-boot:latest  swarmm-agentpool-26830735000000  Shutdown       Complete 23 minutes ago            
nkjtw1wgn4zl   \_ spring-boot.1  ragsns/spring-boot:latest  swarmm-agentpool-26830735000000  Shutdown       Complete 26 minutes ago            
8t4n1a1feeef  spring-boot.2      ragsns/spring-boot:latest  swarmm-agentpool-26830735000004  Running        Running 14 minutes ago             
mrlf2eko75tq  spring-boot.3      ragsns/spring-boot:latest  swarmm-agentpool-26830735000003  Running        Running about a minute ago         
mp8eq4ohrms9  spring-boot.4      ragsns/spring-boot:latest  swarmm-agentpool-26830735000000  Running        Running about a minute ago         
s8b7ic0r9lu4  spring-boot.5      ragsns/spring-boot:latest  swarmm-agentpool-26830735000004  Running        Running about a minute ago         
wkn157yvo2p1  spring-boot.6      ragsns/spring-boot:latest  swarmm-agentpool-26830735000003  Running        Running about a minute ago         
0gexpfftgzb8  spring-boot.7      ragsns/spring-boot:latest  swarmm-agentpool-26830735000003  Running        Running about a minute ago         
4zhs8f8j4540  spring-boot.8      ragsns/spring-boot:latest  swarmm-agentpool-26830735000003  Running        Running about a minute ago         
0esuejv2ktks  spring-boot.9      ragsns/spring-boot:latest  swarmm-agentpool-26830735000000  Running        Running about a minute ago         
b6cgbssis4n0  spring-boot.10     ragsns/spring-boot:latest  swarmm-agentpool-26830735000004  Running        Running about a minute ago
```


If you run the following command a few times

```
curl <ip-of-master>:8080
```

You should notice output from both ther versions indicating that rolling upgrades are happening.

```
Hello World v2!
Hello World v2!
Hello World!
```

Running the following command

```
docker service inspect --pretty spring-boot
```

Will eventually produce an output that looks something like below. Notice that the update is complete.

```

ID:		r4s0zuh99ixd32bumda70d14n
Name:		spring-boot
Service Mode:	Replicated
 Replicas:	10
UpdateStatus:
 State:		completed
 Started:	4 minutes
 Completed:	32 seconds
 Message:	update completed
Placement:
UpdateConfig:
 Parallelism:	1
 On failure:	pause
 Max failure ratio: 0
ContainerSpec:
 Image:		ragsns/spring-boot:v2@sha256:67506fb96a450bf910917b509b299b6e9d617b9eca35b399b65fd16d330a23a9
Resources:
Endpoint Mode:	vip
Ports:
 PublishedPort 8080
  Protocol = tcp
  TargetPort = 8080
```

Now the application has been upgraded to `v2`.

If you run the following command a few times

```
curl <ip-of-master>:8080
```

You should notice output only from the `v2` version.

```
Hello World v2!
Hello World v2!
```


#### Draning a node

If you need to bring an egent down for maintenance or whatever reason you can do that and the framework will readjust to be running the desired instances on the remaining node(s).

Let's run the following command to drain one of the agent nodes. Let's pick a node and call it `DRAIN_NODE`.


```
export DRAIN_NODE=$(docker node ls | grep -v master | tail -1 | awk '{print $2}')
```

Let's get the status of the node via the command

```
docker node inspect --pretty $DRAIN_NODE
```

Which should yield a command that looks something like below indicating the node is available.

```
ID:			mg7sinoolwjkxmyxc6lc79wal
Hostname:		swarmm-agentpool-26830735000003
Joined at:		2017-09-20 00:37:53.731415953 +0000 utc
Status:
 State:			Ready
 Availability:		Active
 Address:		10.0.0.7
Platform:
 Operating System:	linux
 Architecture:		x86_64
Resources:
 CPUs:			2
 Memory:		6.795 GiB
Plugins:
  Network:		bridge, host, macvlan, null, overlay
  Volume:		local
Engine Version:		17.03.2-ce
```

Let's drain the node with the following command

```
docker node update --availability drain $DRAIN_NODE
```

Notice that rerunning the `inspect` command on the node will show a different state from the previous output as below.

```
ID:			mg7sinoolwjkxmyxc6lc79wal
Hostname:		swarmm-agentpool-26830735000003
Joined at:		2017-09-20 00:37:53.731415953 +0000 utc
Status:
 State:			Ready
 Availability:		Drain
 Address:		10.0.0.7
Platform:
 Operating System:	linux
 Architecture:		x86_64
Resources:
 CPUs:			2
 Memory:		6.795 GiB
Plugins:
  Network:		bridge, host, macvlan, null, overlay
  Volume:		local
Engine Version:		17.03.2-ce
```

Running the following command

```
docker service ps spring-boot | grep $DRAIN_NODE
```

Indicates that the different instances running on the node has been redistributed to the rest of the cluster and this node has been taken out of commission from the perspective of the Docker Swarm cluster.
 
```
mrlf2eko75tq   \_ spring-boot.3   ragsns/spring-boot:latest  swarmm-agentpool-26830735000003  Shutdown       Shutdown about an hour ago         
n09l446rn9ko   \_ spring-boot.6   ragsns/spring-boot:v2      swarmm-agentpool-26830735000003  Shutdown       Shutdown 2 minutes ago             
wkn157yvo2p1   \_ spring-boot.6   ragsns/spring-boot:latest  swarmm-agentpool-26830735000003  Shutdown       Shutdown about an hour ago         
gs77nrx8dljj   \_ spring-boot.7   ragsns/spring-boot:v2      swarmm-agentpool-26830735000003  Shutdown       Shutdown 2 minutes ago             
0gexpfftgzb8   \_ spring-boot.7   ragsns/spring-boot:latest  swarmm-agentpool-26830735000003  Shutdown       Shutdown about an hour ago         
xniz8stdu9p7   \_ spring-boot.8   ragsns/spring-boot:v2      swarmm-agentpool-26830735000003  Shutdown       Shutdown 2 minutes ago             
4zhs8f8j4540   \_ spring-boot.8   ragsns/spring-boot:latest  swarmm-agentpool-26830735000003  Shutdown       Shutdown about an hour ago 
```

Bringing the agent back via the following command will make the agent available.

```
docker node update --availability active $DRAIN_NODE
```

Running the following command

```
docker service scale spring-boot=16
```


and rerunning the following command

```
docker service ps spring-boot | grep $DRAIN_NODE
```

Will show some running instances on the agent as below.

```
mrlf2eko75tq   \_ spring-boot.3   ragsns/spring-boot:latest  swarmm-agentpool-26830735000003  Shutdown       Shutdown about an hour ago         
n09l446rn9ko   \_ spring-boot.6   ragsns/spring-boot:v2      swarmm-agentpool-26830735000003  Shutdown       Shutdown 8 minutes ago             
wkn157yvo2p1   \_ spring-boot.6   ragsns/spring-boot:latest  swarmm-agentpool-26830735000003  Shutdown       Shutdown about an hour ago         
gs77nrx8dljj   \_ spring-boot.7   ragsns/spring-boot:v2      swarmm-agentpool-26830735000003  Shutdown       Shutdown 8 minutes ago             
0gexpfftgzb8   \_ spring-boot.7   ragsns/spring-boot:latest  swarmm-agentpool-26830735000003  Shutdown       Shutdown about an hour ago         
xniz8stdu9p7   \_ spring-boot.8   ragsns/spring-boot:v2      swarmm-agentpool-26830735000003  Shutdown       Shutdown 8 minutes ago             
4zhs8f8j4540   \_ spring-boot.8   ragsns/spring-boot:latest  swarmm-agentpool-26830735000003  Shutdown       Shutdown about an hour ago         
pgpgfpve5grr  spring-boot.11      ragsns/spring-boot:v2      swarmm-agentpool-26830735000003  Running        Running 3 seconds ago              
z9htjntfzfi7  spring-boot.12      ragsns/spring-boot:v2      swarmm-agentpool-26830735000003  Running        Running 2 seconds ago              
qrlfg5es3t7b  spring-boot.13      ragsns/spring-boot:v2      swarmm-agentpool-26830735000003  Running        Running 1 second ago               
mngggwdl6l50  spring-boot.14      ragsns/spring-boot:v2      swarmm-agentpool-26830735000003  Running        Running 2 seconds ago              
sr49ydq87qlc  spring-boot.15      ragsns/spring-boot:v2      swarmm-agentpool-26830735000003  Running        Running 2 seconds ago              
8a94mv6j59w1  spring-boot.16      ragsns/spring-boot:v2      swarmm-agentpool-26830735000003  Running        Running 1 second ago 
```

#### Delete the service

Let's go ahead and delete the deployed services via the following command.

```
docker service rm spring-boot
docker service rm nginx
```
### Summary and Next Steps

We started with some simple Docker commands in earlier exercises and used Docker compose but they still don't make it easy to scale and self heal.

Using Docker swarm in swarm mode we're able to meet some of the tenets of an application such as self-healing, scaling, rolling upgrades and so on.

We barely scratched at the surface and you can try out more options provided by the Docker Swarm framework cluster at [https://docs.docker.com/engine/swarm/swarm-tutorial/](https://docs.docker.com/engine/swarm/swarm-tutorial/).

Next, we'll look at another framework that supports thousands of containers referred to as [Kubernetes](../ex5).