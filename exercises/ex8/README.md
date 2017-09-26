# Docker to Helm Hands-On Labs

## Exercise 8: Azure Container Instances

**STOP**! Please ensure that you have met all the prereqisites as mentioned [earlier](../../README.md).

### Azure Container Instances

Although it's necessary to use a framework such as Docker Swarm or Kubernetes for dealing with containers at scale, it may be fine to spin up a single instance of a container relatively quickly. This role is fulfilled by Azure Container instances where you do not need to spin up a VM explicitly.

#### Creating a container

Let's use the `az` CLI to spin up container instances.

Verify that you're logged into Azure and set the right subscription with the following command as mentioned in a [previous execrise](../ex1/README.md).

```
az account show
```

Let's create a Resource Group.

```
az group create --name myaciGroup --location eastus
```

Spinning up an instance is as simple as running the following command.

```
az container create --name spring-boot --image ragsns/spring-boot --cpu 1 --memory 1  --ip-address public -g myaciGroup --port 8080
```

The container will become available and can be verified with the command below.

```
az container show --ids $(az container list | grep id | awk {'print $2'} | sed 's/\"//' | sed 's/\"//' | sed 's/,//')
```

Which will yield an output similar to

```
{
  "containers": [
    {
      "command": null,
      "environmentVariables": [],
      "image": "ragsns/spring-boot",
      "instanceView": {
        "currentState": {
          "detailStatus": "",
          "exitCode": null,
          "finishTime": null,
          "startTime": "2017-09-26T15:09:46+00:00",
          "state": "Running"
        },
        "events": [
          {
            "count": 1,
            "firstTimestamp": "2017-09-26T15:09:03+00:00",
            "lastTimestamp": "2017-09-26T15:09:03+00:00",
            "message": "Pulling: pulling image \"ragsns/spring-boot\"",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2017-09-26T15:09:45+00:00",
            "lastTimestamp": "2017-09-26T15:09:45+00:00",
            "message": "Pulled: Successfully pulled image \"ragsns/spring-boot\"",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2017-09-26T15:09:46+00:00",
            "lastTimestamp": "2017-09-26T15:09:46+00:00",
            "message": "Created: Created container with id b21de874a80ab86897c31bc3384d831c79815d44fc0b5f050749a08ec2e68b38",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2017-09-26T15:09:46+00:00",
            "lastTimestamp": "2017-09-26T15:09:46+00:00",
            "message": "Started: Started container with id b21de874a80ab86897c31bc3384d831c79815d44fc0b5f050749a08ec2e68b38",
            "type": "Normal"
          }
        ],
        "previousState": null,
        "restartCount": 0
      },
      "name": "spring-boot",
      "ports": [
        {
          "port": 8080
        }
      ],
      "resources": {
        "limits": null,
        "requests": {
          "cpu": 1.0,
          "memoryInGb": 1.0
        }
      },
      "volumeMounts": null
    }
  ],
  "id": "/subscriptions/a328d458-73b5-4662-b38d-9d8a16d0fb61/resourceGroups/myaciGroup/providers/Microsoft.ContainerInstance/containerGroups/spring-boot",
  "imageRegistryCredentials": null,
  "ipAddress": {
    "ip": "13.82.235.72",
    "ports": [
      {
        "port": 8080,
        "protocol": "TCP"
      }
    ]
  },
  "location": "eastus",
  "name": "spring-boot",
  "osType": "Linux",
  "provisioningState": "Succeeded",
  "resourceGroup": "myaciGroup",
  "restartPolicy": null,
  "state": "Running",
  "tags": null,
  "type": "Microsoft.ContainerInstance/containerGroups",
  "volumes": null
}
```

Let's grab the public IP address of the container as below.

```
export IP_ADDRESS=$(az container show --ids $(az container list | grep id | awk {'print $2'} | sed 's/\"//' | sed 's/\"//' | sed 's/,//') | grep \"ip\" | awk -F "\"" {'print $4}')
```

Verify that the program is started in the container with the command

```
curl $IP_ADDRESS:8080
```

Let's look at application self healing by exiting the program as below.

```
curl $IP_ADDRESS:8080/exit
```

The program will exit but Azure Container Instances will spawn the program again.

Running the following command again will verify that the program is running and well.

```
curl $IP_ADDRESS:8080
```

Run the following command again and you'll notice in the output that the application terminated and started again.

```
az container show --ids $(az container list | grep id | awk {'print $2'} | sed 's/\"//' | sed 's/\"//' | sed 's/,//')
```

#### Using the Azure Private Repository

So far, we've used the Docker public repository. And Run through the tutorial outlined in [https://docs.microsoft.com/en-us/azure/container-instances/container-instances-tutorial-prepare-app](https://docs.microsoft.com/en-us/azure/container-instances/container-instances-tutorial-prepare-app) for an example of using the Azure private registry which could have been used in earlier examples as well.

#### Delete the Resource Group

Once you're done with this exercise, you can delete the Resource Group with the following command

```
az group delete --name myaciGroup
```

### Summary and Next Steps

We explored Docker, many of the frameworks, including Docker Swarm and Kubernetes. Although Kubernetes is powerful we realized it still had an infrastructure feel to it. Finally, we made a full circle to Azure Container Instances which is probably the simplest way of spinning up a container on Azure.

Congrats, you're now a Docker guru on Azure :-).