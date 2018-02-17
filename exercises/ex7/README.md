# Docker to Helm Hands-On Labs

## Exercise 8: Draft

**STOP**! Please ensure that you have met all the prereqisites as mentioned [earlier](../../README.md) and have created a Kubernetes cluster as outlined in the [previous exercise](../ex5/README.md) and have completed the exercise on [Helm](../ex6/README.md).

### Draft

Draft leverages `helm` and Kubernetes. The focus is on the app rather than infrastructure. 

You can use Draft with any Docker image registry and any Kubernetes cluster, including locally. This exercise leverages the ACS Kubernetes cluster and Azure Container Registry (ACR) to create a live but secure developer pipeline.

Finally, using Azure DNS you can expose that developer pipeline for others to see at a domain.

Let's start by installing draft on the cluster as outlined in [https://github.com/Azure/draft/blob/master/docs/install.md](https://github.com/Azure/draft/blob/master/docs/install.md).

### Verify the version

Verify that the version is later than 0.8.0 by running the following command.

```
draft version
```

Which should produce an output that looks something like below.

```
Client: &version.Version{SemVer:"v0.8.0", GitCommit:"6edb87f6e85d019fc3f708d31ca76270b765a9b0", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v0.8.0", GitCommit:"6edb87f6e85d019fc3f708d31ca76270b765a9b0", GitTreeState:"clean"}
```
### Draft Samples

Git clone draft repository which includes the samples using the following command.

```
git clone https://github.com/Azure/draft/

```
####  Create an Azure Container registry

Start by creating a resource group. You may have to specify an unique name.

```
az acr create --resource-group draft --name draftacs --sku Basic --admin-enabled true
```

Create a Azure Container Registry with the following command. Substitute `ragsns` with your own unique name.

```
az acr create --resource-group draft --name ragsns --sku Basic --admin-enabled true
```

You'll get an output that looks something like below.

```
A new storage account 'ragsns180830' will be created in resource group 'draft'.

Create a new service principal and assign access:
  az ad sp create-for-rbac --scopes /subscriptions/a328d458-73b5-4662-b38d-9d8a16d0fb61/resourceGroups/draft/providers/Microsoft.ContainerRegistry/registries/ragsns --role Owner --password <password>

Use an existing service principal and assign access:
  az role assignment create --scope /subscriptions/a328d458-73b5-4662-b38d-9d8a16d0fb61/resourceGroups/draft/providers/Microsoft.ContainerRegistry/registries/ragsns --role Owner --assignee <app-id>
{
  "adminUserEnabled": true,
  "creationDate": "2017-09-26T18:08:53.801270+00:00",
  "id": "/subscriptions/a328d458-73b5-4662-b38d-9d8a16d0fb61/resourceGroups/draft/providers/Microsoft.ContainerRegistry/registries/ragsns",
  "location": "eastus",
  "loginServer": "ragsns.azurecr.io",
  "name": "ragsns",
  "provisioningState": "Succeeded",
  "resourceGroup": "draft",
  "sku": {
    "name": "Basic",
    "tier": "Basic"
  },
  "storageAccount": {
    "name": "ragsns180830"
  },
  "tags": {},
  "type": "Microsoft.ContainerRegistry/registries"
}

```

Get the password with the following command, substituting `ragsns` with your own unique registry name.

```
az acr credential show -n ragsns --query passwords\[0\].value
```

You'll use this password in the next step.

#### Initialize draft

Initialize `draft` using the following command.

```
draft init
```

and provide the values substituting the credentials for your unique repository, and a top-level domain (don't worry if you don't have a domain. Any value is fine here) including the password you just obtained.

```
In order to install Draft, we need a bit more information...

1. Enter your Docker registry URL (e.g. docker.io, quay.io, myregistry.azurecr.io): ragsns.azurecr.io
2. Enter your username: ragsns
3. Enter your password: 
Draft has been installed into your Kubernetes Cluster.
Happy Sailing!
```

Let's look at the nodejs example by traversing into that sub-directory.

```
cd draft/examples/example-nodejs
```

and invoking the following command

```
draft create
```

which will generate an output that looks like below

```
draft create
--> Draft detected the primary language as JavaScript with 63.847430% certainty.
--> Ready to sail
```

Run the following command to see what it auto-generated based on the language it detected.

```
ls -l
```

Edit `index.js` and change the line with the `Hello World` entry to look something like below.

```
  response.end("Hello World v1.0, I am Node.js!");
```

Now run the following command to stand up the app, which builds the dockerized image, pushes to the Docker repository and stands it up as a Helm app. 

```
draft up
```

If everything goes well, you'll see an output that builds the Docker image and publishes it with an output that will end as below.

```
SUCCESS ⚓  (5.3008s)
olfactory-bronco: Build ID: 01BXCPYB3036FP1EBSFHEYNBJM
```

Run the status command on the app as below. In this case it is `olfactory-bronco`.

```
helm status <application-name>
```

### Connect to the application

Run the following command

```
draft connect
```

This will yield an output something like below.

```
Connecting to your app...SUCCESS...
Connect to your app on localhost:56512
Starting log streaming...

> example-nodejs@0.0.0 start /usr/src/app
> node index.js

server is listening on 8080

```

You can connect to the application based on the output of the command above using the approriate port on `localhost`. In this case it is `56512`.

```
curl localhost:56512
```

Change the line in `index.js` to look like below.

```
  response.end("Hello World v2.0, I am Node.js!");
```

Again, invoke the command

```
draft up
```
 
Followed by the command.

```
draft connect
```

Note the output from v2 of the application and run the following command

```
curl localhost:<port-from-output-from-draft-connect>
```

```
Hello World v2.0, I am Node.js!
```

Let's look at the application history with the following command

```
helm history listening-numbat
```

Which should yield an output that looks something like below.

```
REVISION	UPDATED                 	STATUS    	CHART                 	DESCRIPTION     
1       	Tue Sep 26 14:36:09 2017	SUPERSEDED	listening-numbat-0.1.0	Install complete
2       	Tue Sep 26 14:38:08 2017	DEPLOYED  	listening-numbat-0.1.0	Upgrade complete
```

#### Rollback

We are going to rollback to the previous version, using the following command

```
helm rollback listening-numbat 1
```

You should see an output that looks something like below.

```
Rollback was a success! Happy Helming!
```

If you run the following command again

```
curl --header Host:listening-numbat.rags.tech 52.170.208.124
```

You'll see the output from the rolled back version as below.

```
Hello World v1.0, I am Node.js!
```

Rrerunning the following command

```
helm history listening-numbat
```

Will produce an output confirming it

```                            
REVISION	UPDATED                 	STATUS    	CHART                 	DESCRIPTION     
1       	Tue Sep 26 14:36:09 2017	SUPERSEDED	listening-numbat-0.1.0	Install complete
2       	Tue Sep 26 14:38:08 2017	SUPERSEDED	listening-numbat-0.1.0	Upgrade complete
3       	Tue Sep 26 14:40:41 2017	DEPLOYED  	listening-numbat-0.1.0	Rollback to 1
```

#### Setting up a top level domain (optional)

You could setup a top level DNS domain as outlined in [https://docs.microsoft.com/en-us/azure/container-service/kubernetes/container-service-draft-up](https://docs.microsoft.com/en-us/azure/container-service/kubernetes/container-service-draft-up).

### Setup top level domain via draft

Run the following command to delete draft artifacts in helm as below.

```
helm del --purge draft
```
### Setup the Ingress

Run the following command

```
draft init --ingress-enabled
```

and provide the values as before with the top level domain, in this case `rags.tech`.

You can get the applications running using the command below.

```
helm list
```
Now get the status of the running application as below.

```
helm status <application-name>
```

Which should yield an output that looks something like below.

``` 
LAST DEPLOYED: Thu Oct 26 23:02:43 2017
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Service
NAME                         CLUSTER-IP   EXTERNAL-IP  PORT(S)   AGE
olfactory-bronco-javascript  10.0.116.39  <none>       8080/TCP  15m

==> v1beta1/Deployment
NAME                         DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
olfactory-bronco-javascript  2        2        2           2          15m

==> v1beta1/Ingress
NAME                         HOSTS                       ADDRESS  PORTS  AGE
olfactory-bronco-javascript  olfactory-bronco.rags.tech  80       15m


NOTES:

  http://olfactory-bronco.rags.tech to access your application
```

Based on DNS, the application end point can be accessed directly as below.

```
curl http://olfactory-bronco.rags.tech
```

### Summary and Next Steps

We started with some simple Docker commands in earlier exercises and used Docker compose but they still don't make it easy to scale and self heal.

Using Docker swarm in swarm mode we're able to meet some of the tenets of an application such as self-healing, scaling, rolling upgrades and so on.

We used Kubernetes as an orchestrator to essentially accomplish the same things we did with Dockder Swarm.

Helm made it a lot easier to install kubernetes apps on the cluster.

Draft takes it further, leverages Helm (and Kubernetes) to provide a higher level of abstraction as an application.

In the [Next Exercise](../ex8/README.md) we pivot to an different way and much simpler way of spinning up containers using Azure Container Instances.