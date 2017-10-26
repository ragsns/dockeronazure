# Docker to Helm Hands-On Labs

## Exercise 5: Kubernetes

**STOP**! Please ensure that you have met all the prereqisites as mentioned [earlier](../../README.md).

### Kubernetes

We saw in the previous exercise that Azure Container Service (ACS) provides a way to simplify the creation, configuration, and management of a cluster of virtual machines that are preconfigured to run containerized applications. 

According to [https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/) the goal of Kubernetes is to

- Deploy applications quickly and predictably.
- Scale applications on the fly.
- Roll out new features seamlessly.
- Limit hardware usage to required resources only.

Which we saw how we could accomplish in the previous exercise. Kubernetes can do this on thousands of applications running in containers.

### Creating a Kubernetes cluster on Azure

Let's use the `az` CLI to spin up a Kubernetes cluster.

Verify that you're logged into Azure and set the right subscription with the following command as mentioned in a [previous execrise](../ex1/README.md).

```
az account show
```

We create the cluster using the following commands. You could do this from the Azure portal as well. (**unique names (USE INITIALS for Resource Group, Cluster Name and DNS prefix might be required**). 


```
export DNS_PREFIX=kub07
export CLUSTER_NAME=kubclus07
export RESOURCE_GROUP=kub-rg-07 
```

You may have make these names unique if the cluster creation fails.

Let's start with creating a resource group providing a name for a Resource Group.

```
az group create --name $RESOURCE_GROUP --location eastus 

```

Next create the Kubernetes cluster with the following command 
```
az acs create --orchestrator-type=kubernetes --resource-group $RESOURCE_GROUP --name=$CLUSTER_NAME --dns-prefix=$DNS_PREFIX --generate-ssh-keys
```

To manage the Kubernetes cluster, use `kubectl`, the Kubernetes command-line client.

If you're using Azure CloudShell, `kubectl` is already installed. If you want to install it locally, like on your laptop, you can use the  following command

```
az acs kubernetes install-cli
```

To configure kubectl to connect to your Kubernetes cluster, run the following command.

```
az acs kubernetes get-credentials --resource-group=$RESOURCE_GROUP --name=$CLUSTER_NAME
```

This step downloads credentials and configures the Kubernetes CLI to use them.

To verify the connection to your cluster, use the kubectl get command to return a list of the cluster nodes.

```
kubectl get nodes
```

which will yield an output that looks something like below.

```
NAME                    STATUS                     AGE       VERSION
k8s-agent-5bcaf905-0    Ready                      43d       v1.6.6
k8s-agent-5bcaf905-1    Ready                      43d       v1.6.6
k8s-agent-5bcaf905-2    Ready                      43d       v1.6.6
k8s-master-5bcaf905-0   Ready,SchedulingDisabled   43d       v1.6.6
```

You can get the cluster information with the following command.

```
kubectl cluster-info
```

which should yield an output something like below.

```
Kubernetes master is running at https://kub07mgmt.eastus.cloudapp.azure.com

Heapster is running at https://kub07mgmt.eastus.cloudapp.azure.com/api/v1/namespaces/kube-system/services/heapster/proxy

KubeDNS is running at https://kub07mgmt.eastus.cloudapp.azure.com/api/v1/namespaces/kube-system/services/kube-dns/proxy

kubernetes-dashboard is running at https://kub07mgmt.eastus.cloudapp.azure.com/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy

tiller-deploy is running at https://kub07mgmt.eastus.cloudapp.azure.com/api/v1/namespaces/kube-system/services/tiller-deploy/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

### Create a Deployment

To create the deployment, use the following command.

```
kubectl run spring-boot --image=ragsns/spring-boot --port=8080
```


### Create a Proxy

Create a proxy with the following command.

```
kubectl proxy
```

which will yield an output that looks something like below.

```
Starting to serve on 127.0.0.1:8001
```

Run the following command
```
curl localhost:8001
```
###

Which will yield an output that looks something like below which lists the endpoints of the Kubernetes cluster.

```
{

  "paths": [

    "/api",

    "/api/v1",

    "/apis",

    "/apis/apps",

    "/apis/apps/v1beta1",

    "/apis/authentication.k8s.io",

    "/apis/authentication.k8s.io/v1",

    "/apis/authentication.k8s.io/v1beta1",

    "/apis/authorization.k8s.io",

    "/apis/authorization.k8s.io/v1",

    "/apis/authorization.k8s.io/v1beta1",

    "/apis/autoscaling",

    "/apis/autoscaling/v1",

    "/apis/autoscaling/v2alpha1",

    "/apis/batch",

    "/apis/batch/v1",

    "/apis/batch/v2alpha1",

    "/apis/certificates.k8s.io",

    "/apis/certificates.k8s.io/v1beta1",

    "/apis/extensions",

    "/apis/extensions/v1beta1",

    "/apis/policy",

    "/apis/policy/v1beta1",

    "/apis/rbac.authorization.k8s.io",

    "/apis/rbac.authorization.k8s.io/v1alpha1",

    "/apis/rbac.authorization.k8s.io/v1beta1",

    "/apis/settings.k8s.io",

    "/apis/settings.k8s.io/v1alpha1",

    "/apis/storage.k8s.io",

    "/apis/storage.k8s.io/v1",

    "/apis/storage.k8s.io/v1beta1",

    "/healthz",

    "/healthz/ping",

    "/healthz/poststarthook/bootstrap-controller",

    "/healthz/poststarthook/ca-registration",

    "/healthz/poststarthook/extensions/third-party-resources",

    "/logs",

    "/metrics",

    "/swaggerapi/",

    "/ui/",

    "/version"

  ]

}
```

Kubernetes uses Pods as deployment units and we'll get the pod name as below.

```
export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}'| grep spring-boot)
echo Name of the Pod: $POD_NAME
```

This will yield the pod name as below.

```
name of the pod: spring-boot-178131612-nk38z
```

To verify the service is running, run the following command.

```
curl http://localhost:8001/api/v1/proxy/namespaces/default/pods/$POD_NAME/
```

which should yield the output from the service as below.

```
Hello World!
```

Now run the following command

```
kubectl get pods | grep -i spring-boot
```

As we did in the previous exercise, we will terminate the program with the following command.

```
curl http://localhost:8001/api/v1/proxy/namespaces/default/pods/$POD_NAME/exit
```

which will yield an output that looks something like below.

```
Error: 'EOF'

Trying to reach: 'http://10.244.0.27:8080/exit'
```

Let's try the earlier command a few times

```
kubectl get pods | grep -i spring-boot
```

which might yield the following output showing the app terminating and restarting.

```
spring-boot-1386092105-jbgtx                                    0/1       Completed          1          51m
spring-boot-1386092105-jbgtx                                    1/1       Running            2          51m
```

You can get the logs of the running application using

```
kubectl logs $POD_NAME
```

If necessary, you can login to the Kubernetes master using the following command substituting the region name if required.

```
ssh azureuser@$(echo $DNS_PREFIX)mgmt.eastus.cloudapp.azure.com
```

From the local laptop, we expose the deployment with the following command.

```
kubectl expose deployment/spring-boot --type="NodePort" --port 8080
```

which should yield the following command.
```
service "spring-boot" exposed
```

Now, let's get more information on this using the following command.

```
kubectl describe services/spring-boot | grep -i nodeport | grep -i tcp
```

which should yield an output that looks something like below.

```
Type:			NodePort
NodePort:		<unset>	30961/TCP
```

Let's get the output of the program using the following command

```
NODE_PORT=$(kubectl describe services/spring-boot | grep -i nodeport | grep -i tcp | awk '{print $3}' | sed 's/\/TCP//')

curl localhost:$NODE_PORT
```

### Scaling the service

Scale the service using the following command

```
kubectl scale deployments/spring-boot --replicas=4
```

Running the following command

```
kubectl get pods | grep -i spring-boot
```

will yield the pods running the service as shown below.

```
spring-boot-1386092105-fb6w1                                    1/1       Running            0          7m
spring-boot-1386092105-fvl06                                    1/1       Running            2          7m
spring-boot-1386092105-jbgtx                                    1/1       Running            2          1h
spring-boot-1386092105-zd6lp                                    1/1       Running            0          7m
```

### Application Self-Healing

Terminating an instance of the service with the command below

```
curl localhost:$NODE_PORT/exit
```

indicates termination as below

```
curl: (52) Empty reply from server
```

Running the following command 

```
kubectl get deployments | grep -i spring-boot
```

shows the number of running instances to 3 and eventually back to 4.

```
spring-boot                                    4         4         4            3           1h
spring-boot                                    4         4         4            4           1h
```

### Upgrade the Service

The service can be upgraded as below

```
kubectl set image deployments/spring-boot spring-boot=ragsns/spring-boot:v2
```

and run the following command **immediately** 

```
kubectl rollout status deployments/spring-boot
```
which yields an output something like below showing the status of the rolling upgrade.

```
Waiting for rollout to finish: 1 old replicas are pending termination...
Waiting for rollout to finish: 1 old replicas are pending termination...
Waiting for rollout to finish: 1 old replicas are pending termination...
deployment "spring-boot" successfully rolled out

```

Running the command below verifies that the service was updated

```
curl localhost:$NODE_PORT

```

with the output from the updated service as below.

```
Hello World v2!
```

### Rolling back the upgrades

You can rollback the upgrade using the following command

```
kubectl rollout undo deployments/spring-boot
```

Running the following command again

```
curl localhost:$NODE_PORT
```

which yields the following output indicating the service was rolled back.

```
Hello World!
```


### Clean up


To delete the deployment, run the following command.

```
kubectl delete deploy/spring-boot
```

To delete the service, run the following command.

```
kubectl delete svc spring-boot
```

### Summary and Next Steps

We started with some simple Docker commands in earlier exercises and used Docker compose but they still don't make it easy to scale and self heal.

Using Docker swarm in swarm mode we're able to meet some of the tenets of an application such as self-healing, scaling, rolling upgrades and so on.

We used Kubernetes as an orchestrator in this exercise to essentially accomplish the same things we did in the previous exercise.

More Kubernetes command information is at [https://kubernetes.io/docs/tutorials/kubernetes-basics/](https://kubernetes.io/docs/tutorials/kubernetes-basics/). You don't have to worry about the `minikube` part of the tutorial since a cluster was spun up on Azure.

Kubernetes still seems more like infrastructure and we'll look at [Helm and Draft](../ex6) which provides a more application perspective.