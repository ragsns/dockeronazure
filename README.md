# Docker on Azure Hands-On Lab - Docker to Helm

## Prerequisites

Needless to say you'll need a laptop! Any OS is fine (Mac is recommended and that's the only machine we've tested on) but make sure the following prerequisites are met prior to the session:

- **Prerequisite 1:** [Git](http://git-scm.com/downloads) is installed

- **Prerequisite 2:** A valid Azure subscription is available. You can get a trial account from [https://azure.microsoft.com/en-us/free/](https://azure.microsoft.com/en-us/free/).

- **Prerequisite 3:** The `az` CLI from [https://github.com/Azure/azure-cli] (https://github.com/Azure/azure-cli) is installed on the laptop **OR** you can use the cloud shell as outlined in [https://docs.microsoft.com/en-us/azure/cloud-shell/overview] (https://docs.microsoft.com/en-us/azure/cloud-shell/overview)

- **Prerequisite 4:** Limits in Azure are set low. File support tickets to get the limits bumped up for your subscriptions. We will need cores (60-100 recommended) to be spun up in West UK which supports some of the configurations.

- **Prerequiste 5** The labs from [https://github.com/ragsns/dockeronazure.git](https://github.com/ragsns/dockeronazure.git) installed **locally** either via a `git clone` command or by downloading a zip file from the remote repository.


## Samples and General Directions

Each directory is in a separate sub-directory. ***Ensure that you're in the sub-directory when you're working on a particular exercise and you're issuing the CLI commands from the subdirectory pertaining to the exercise.***


## Recommended Exercises - User Related

It is recommended that you run through these exercises sequentially since they are progressive with some dependencies. Each exercise should take about 10-20 mins. to complete.

- Exercise 1: [Login to Azure](exercises/ex1)
- Exercise 2: [Gentle Intro to Docker](exercises/ex2)
- Exercise 3: [Docker Compose](exercises/ex3)
- Exercise 4: [Docker Swarm](exercises/ex4)
- Exercise 5: [Kubernetes or K8S](exercises/ex5)
- Exercise 6: [Helm](exercises/ex6) (optional)
- Exercise 7: [Draft](exercises/ex7) (optional)
- Exercise 8: [Container Instances](exercises/ex8)

## More Resources

Docker Getting Started Material at [https://docs.docker.com/get-started/](https://docs.docker.com/get-started/)

Kubernetes at [https://kubernetes.io/docs/tutorials/kubernetes-basics/](https://kubernetes.io/docs/tutorials/kubernetes-basics/)

Kubernetes on Azure at [https://blogs.msdn.microsoft.com/azureedu/2017/04/23/how-can-i-get-started-with-kubernetes-on-azure-container-services/](https://blogs.msdn.microsoft.com/azureedu/2017/04/23/how-can-i-get-started-with-kubernetes-on-azure-container-services/)

Helm getting started at [https://docs.helm.sh/using_helm/#quickstart](https://docs.helm.sh/using_helm/#quickstart)

Draft getting started at [https://github.com/Azure/draft](https://github.com/Azure/draft)

Azure getting started at [https://azure.microsoft.com/en-us/get-started/](https://azure.microsoft.com/en-us/get-started/)


##Contact

Please contact me on Twitter @ragss.
