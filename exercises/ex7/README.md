# Docker to Helm Hands-On Labs

## Exercise 8: Draft

**STOP**! Please ensure that you have met all the prereqisites as mentioned [earlier](../../README.md) and have created a Kubernetes cluster as outlined in the [previous exercise](../ex5/README.md) and have completed the exercise on [Helm](../ex6/README.md).

### Draft

Draft leverages `helm` and Kubernetes. Let's start by installing draft on the cluster as outlined in [https://github.com/Azure/draft/blob/master/docs/install.md](https://github.com/Azure/draft/blob/master/docs/install.md).


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

Initialize `draft' using the following command.
```
draft init
```

and provide the values substituting the credentials for your unique repository, and a top-level domain (don't worry if you don't have a domain. Any value is fine here) including the password you just obtained.

```
In order to install Draft, we need a bit more information...

1. Enter your Docker registry URL (e.g. docker.io, quay.io, myregistry.azurecr.io): ragsns.azurecr.io
2. Enter your username: ragsns
3. Enter your password: 
4. Enter your org where Draft will push images [ragsns]: 
5. Enter your top-level domain for ingress (e.g. draft.example.com): rags.tech
Draft has been installed into your Kubernetes Cluster.
Happy Sailing!
```

Let's look at the nodejs example by traversing into that sub-directory.

```
cd draft/examples/nodejs
```

and invoking the following command

```
draft create
```

which will generate an output that looks like below

```
--> Node.js app detected
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

Now run the following command to stand up the app, which builds the dockerized image, pushes to the Docker repository and stands it up as a Helm app. If everything goes well, you'll see an output that looks something like below.

```
--> Deploying to Kubernetes
    Release "listening-numbat" does not exist. Installing it now.
--> Status: DEPLOYED
--> Notes:
     
  http://listening-numbat.rags.tech to access your application
```

Hit `ctrl-c`. Run the status command on the app as below. In this case it is `listening-numbat`.

```
helm status <application-name>
```

Now run the following command substituting the external IP of the ingress controller that was referred to as `external-ip-of-ingress-controller` in the previous exercise as below.

```
curl --header Host:listening-numbat.rags.tech 52.170.208.124 # use <external-ip-of-ingress-controller>
```
This will yield an output something like below.

```
Hello World v1.0, I am Node.js!
```

Change the line in `index.js` to look like below.

```
  response.end("Hello World v2.0, I am Node.js!");
```

Again, invoke the command

```
draft up
```

Again, run the command substituting the public IP of the nginx ingress.

```
curl --header Host:listening-numbat.rags.tech 52.170.208.124
```

Note the output from v2 of the application as below.

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

We are going to rollback to the previous version, jjust because we can using the following command

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

### Summary and Next Steps

We started with some simple Docker commands in earlier exercises and used Docker compose but they still don't make it easy to scale and self heal.

Using Docker swarm in swarm mode we're able to meet some of the tenets of an application such as self-healing, scaling, rolling upgrades and so on.

We used Kubernetes as an orchestrator to essentially accomplish the same things we did with Dockder Swarm.

Helm made it a lot easier to install kubernetes apps on the cluster.

Draft takes it further, leverages Helm (and Kubernetes) to provide a higher level of abstraction as an application.

In the [Next Exercise](../ex8/README.md) we pivot to a different way of spinning up containers using Azure Container Instances.