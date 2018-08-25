# Docker to Helm Hands-On Labs

## Exercise 7: Draft

**STOP**! Please ensure that you have met all the prereqisites as mentioned [earlier](../../README.md) and have created a Kubernetes cluster as outlined in the [previous exercise](../ex5/README.md) and have completed the exercise on [Helm](../ex6/README.md).

### Draft

Draft leverages `helm` and Kubernetes. The focus is on the app rather than infrastructure. 

You can use Draft with any Docker image registry and any Kubernetes cluster, including locally. This exercise leverages the ACS Kubernetes cluster and Azure Container Registry (ACR) to create a live but secure developer pipeline.

Let's start by installing draft on the cluster as outlined in [https://github.com/Azure/draft/blob/master/docs/install.md](https://github.com/Azure/draft/blob/master/docs/install.md).

### Verify the version

Verify that the version is later than 0.8.0 by running the following command.

```
draft version
```

Which should produce an output that looks something like below.

```
&version.Version{SemVer:"v0.15.0", GitCommit:"9d73889a1318a435d126bc5df846610d30cfbe7f", GitTreeState:"clean"}
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
```

Create a new service principal and assign access:

```
  az ad sp create-for-rbac --scopes /subscriptions/a328d458-73b5-4662-b38d-9d8a16d0fb61/resourceGroups/draft/providers/Microsoft.ContainerRegistry/registries/ragsns --role Owner --password <password>
```

Use an existing service principal and assign access:

```
  az role assignment create --scope /subscriptions/a328d458-73b5-4662-b38d-9d8a16d0fb61/resourceGroups/draft/providers/Microsoft.ContainerRegistry/registries/ragsns --role Owner --assignee <app-id>
```

You should see an output that looks something like below.

```
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
You should see an output that looks something like below.

```
Installing default plugins...
Installation of default plugins complete
Installing default pack repositories...
Installation of default pack repositories complete
$DRAFT_HOME has been configured at /Users/raghavansrinivas/.draft.
Happy Sailing!
```

#### Configure Draft to push to and deploy from ACR

Set the Draft configuration registry value after replacing the `acrName` in the following command.

```
draft config set registry <acrName>.azurecr.io
```

Login to ACR with the following command substituting `acrName`. 

```
az acr login --name <acrName>
```

This will create a trust between AKS and ACR. As a result, no passwords or secrets are required to push to or pull from the ACR registry. Authentication happens at the Azure Resource Manager level, using Azure Active Directory.

#### Pushing an example

Let's look at the nodejs example by traversing into that sub-directory.

```
cd examples/example-nodejs
```

and invoking the following command

```
draft create
```

which will generate an output that looks like below

```
draft create
--> Draft detected JavaScript (58.280255%)
--> Ready to sail
```

Run the following command to see that it auto-generated a `charts` sub-directory among other things that contains the chart used to deploy the application, based on the language it detected.

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
Draft Up Started: 'example-nodejs': 01CNQG7G8E1CQJKS5JJ7T7S1RA
example-nodejs: Building Docker Image: SUCCESS ⚓  (85.0426s)
example-nodejs: Pushing Docker Image: SUCCESS ⚓  (593.3160s)
example-nodejs: Releasing Application: SUCCESS ⚓  (3.9282s)
Inspect the logs with `draft logs 01CNQG7G8E1CQJKS5JJ7T7S1RA`
```

Run the status command on the app as below. In this case it is `example-nodejs`.

```
helm status example-nodejs
```

### Connect to the application

Run the following command

```
draft connect
```

This will yield an output something like below.

```
Connect to javascript:8080 on localhost:51592
[javascript]: 
[javascript]: > example-nodejs@0.0.0 start /usr/src/app
[javascript]: > node index.js
[javascript]: 
[javascript]: server is listening on 8080
[javascript]: /
```

From another terminal window, you can connect to the application based on the output of the command above using the approriate port on `localhost`. In this case it is `51592`.

```
curl localhost:51592
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

Note the output from v2 of the application and run the following command from another terminal window

```
curl localhost:<port-from-output-from-draft-connect>
```

```
Hello World v2.0, I am Node.js!
```

Let's look at the application history with the following command

```
helm history example-nodejs
```

Which should yield an output that looks something like below.

```
REVISION	UPDATED                 	STATUS    	CHART            	DESCRIPTION     
1       	Fri Aug 24 23:20:54 2018	SUPERSEDED	javascript-v0.1.0	Install complete
2       	Fri Aug 24 23:22:10 2018	DEPLOYED  	javascript-v0.1.0	Upgrade complete
```

#### Rollback

We are going to rollback to the previous version, using the following command

```
helm rollback example-nodejs 1
```

You should see an output that looks something like below.

```
Rollback was a success! Happy Helming!
```

If you connect to the application again with the following command.

```
draft connect
```

and `curl` the appropriate port

```
curl localhost:52414
```

You'll see the output from the rolled back version as below.

```
Hello World v1.0, I am Node.js!
```

Rrerunning the following command

```
helm history example-nodejs
```

Will produce an output confirming it

```                            
REVISION	UPDATED                 	STATUS    	CHART            	DESCRIPTION     
1       	Fri Aug 24 23:38:57 2018	SUPERSEDED	javascript-v0.1.0	Install complete
2       	Fri Aug 24 23:39:45 2018	SUPERSEDED	javascript-v0.1.0	Upgrade complete
3       	Fri Aug 24 23:41:17 2018	DEPLOYED  	javascript-v0.1.0	Rollback to 1
```

### Summary and Next Steps

We started with some simple Docker commands in earlier exercises and used Docker compose but they still don't make it easy to scale and self heal.

Using Docker swarm in swarm mode we're able to meet some of the tenets of an application such as self-healing, scaling, rolling upgrades and so on.

We used Kubernetes as an orchestrator to essentially accomplish the same things we did with Docker Swarm.

Helm made it a lot easier to install kubernetes apps on the cluster.

Draft takes it further, leverages Helm (and Kubernetes) to provide a higher level of abstraction as an application.

In the [Next Exercise](../ex8/README.md) we pivot to an different way and much simpler way of spinning up containers using Azure Container Instances.