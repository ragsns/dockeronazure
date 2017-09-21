# Docker to Helm Hands-On Labs

## Exercise 1: Login to Azure

**STOP** Please ensure that you have met all the prereqisites as mentioned [earlier] (../../README.md).

Login to Azure via the CLI. Instructions are at [https://docs.microsoft.com/en-us/azure/xplat-cli-connect](https://docs.microsoft.com/en-us/azure/xplat-cli-connect).

After successful login issue the following command

```
az account list
```

```
[
  {
    "cloudName": "AzureCloud",
    "id": "<id 1>",
    "isDefault": false,
    "name": "Visual Studio Enterprise",
    "state": "Enabled",
    "tenantId": "<tenant 1>",
    "user": {
      "name": "<name>@microsoft.com",
      "type": "user"
    }
  },
  {
    "cloudName": "AzureCloud",
    "id": "<id 2>",
    "isDefault": true,
    "name": "AIRS",
    "state": "Enabled",
    "tenantId": "<tenant 2>",
    "user": {
      "name": "<name>@microsoft.com",
      "type": "user"
    }
  }
]
```
### Ensure that you set the right subscription

If you have multiple subscriptions, issue the following command to make sure you set the right subscription.

```
az account set --subscription <correct id>
```

### Ensure that container instances are supported

Verify that the container instance component is supported in the CLI by running the following command

```
az container list
```

which should yield the following

```
[]
```
or the running containers if you have them running already.

### Install kubectl (if required)

The command `kubectl` is used to manage the Kubernetes (K8s) cluster. This is already installed in the cloud shell. 

If you're using the CLI from your laptop, we will be using the `kubectl` command later. But, let's go ahead and install it right now with the following command

```
az acs kubernetes install-cli
```

### Ensure that you are in the right sub-directory

Ensure that you are in sub-directory ex1.

```
cd <path-to-hol-folder>/dockeronazure/exercises/ex1
```

You can also try additional options for the `az` CLI.
