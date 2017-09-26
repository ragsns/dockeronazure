# Docker to Helm Hands-On Labs

## Exercise 6: Helm

**STOP**! Please ensure that you have met all the prereqisites as mentioned [earlier](../../README.md) and have created a Kubernetes cluster as outlined in the [previous exercise](../ex5/README.md).

### Helm

Install Helm as outlined in [https://github.com/kubernetes/helm#install](https://github.com/kubernetes/helm#install). You can use the following commands

```
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
```

### Initialize the Helm client and server

Initialize `helm` with the following command

```
helm init
```

This will install the server component referred to as `tiller` as well. Verify that this is installed with the following command.

```
kubectl --namespace kube-system get pods | grep tiller
```

Which should indicate that it's running on the cluster as below.

```
tiller-deploy-3019006398-2656f                                1/1       Running       0          5d 
```

Helm uses the concepts of charts for deployment of the Helm packages. Think of Helm as the `apt-get` for Kubernetes.


#### Installing a Helm application

You can search for Helm applications using the following command

```
helm search
```

Let's install `dokuwiki` with the following command

```
helm install stable/dokuwiki
```

Which will produce an output that looks something like below.

```
NAME:   ordered-greyhound
LAST DEPLOYED: Tue Sep 26 01:49:08 2017
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Service
NAME                        CLUSTER-IP   EXTERNAL-IP  PORT(S)                     AGE
ordered-greyhound-dokuwiki  10.0.220.48  <pending>    80:31914/TCP,443:31530/TCP  5s

==> v1beta1/Deployment
NAME                        DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
ordered-greyhound-dokuwiki  1        1        1           0          5s

==> v1/Secret
NAME                        TYPE    DATA  AGE
ordered-greyhound-dokuwiki  Opaque  1     5s

==> v1/PersistentVolumeClaim
NAME                                 STATUS  VOLUME                                    CAPACITY  ACCESSMODES  STORAGECLASS  AGE
ordered-greyhound-dokuwiki-dokuwiki  Bound   pvc-6fa00539-a27e-11e7-b583-000d3a102f94  8Gi       RWO          default       5s
ordered-greyhound-dokuwiki-apache    Bound   pvc-6fa22a2b-a27e-11e7-b583-000d3a102f94  1Gi       RWO          default       5s


NOTES:

** Please be patient while the chart is being deployed **

1. Get the DokuWiki URL by running:

** Please ensure an external IP is associated to the ordered-greyhound-dokuwiki service before proceeding **
** Watch the status using: kubectl get svc --namespace default -w ordered-greyhound-dokuwiki **

  export SERVICE_IP=$(kubectl get svc --namespace default ordered-greyhound-dokuwiki -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
  echo http://$SERVICE_IP/

2. Login with the following credentials

  echo Username: user
  echo Password: $(kubectl get secret --namespace default ordered-greyhound-dokuwiki -o jsonpath="{.data.dokuwiki-password}" | base64 --decode)
```

Let's watch the public IP being generated with a command that you can cut-n-paste from output above.

```
kubectl get svc --namespace default -w ordered-greyhound-dokuwiki
```

Which will yield the public IP address of the service running on the cluster as seen below.

```
NAME                         CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
ordered-greyhound-dokuwiki   10.0.220.48   <pending>     80:31914/TCP,443:31530/TCP   4m
ordered-greyhound-dokuwiki   10.0.220.48   52.170.36.159   80:31914/TCP,443:31530/TCP   5m
```

Run the following command substituting the external IP address based on output above

```
curl http://<external-ip>/doku.php
```

The above command will generate the output from the service.

Let's also `helm` install `nginx-ingress` based on the command below.

```
helm install stable/nginx-ingress --namespace=default --name=nginx2-ingress
```

You can watch the external IP is generated using the following command

```
kubectl get svc --namespace default -w nginx2-ingress-nginx-ingress-controller
```

Which will eventually generate an external IP as below.

```
NAME                                      CLUSTER-IP   EXTERNAL-IP    PORT(S)                      AGE
nginx2-ingress-nginx-ingress-controller   10.0.92.72   52.170.208.124  80:30626/TCP,443:30763/TCP   42d
```

We will refer to this as `external-ip-of-ingress-controller`.

### Summary and Next Steps

We started with some simple Docker commands in earlier exercises and used Docker compose but they still don't make it easy to scale and self heal.

Using Docker swarm in swarm mode we're able to meet some of the tenets of an application such as self-healing, scaling, rolling upgrades and so on.

We used Kubernetes as an orchestrator in the previous exercise to essentially accomplish the same things we did with Dockder Swarm.

More Helm command information is at [https://docs.bitnami.com/kubernetes/how-to/deploy-application-kubernetes-helm/](https://docs.bitnami.com/kubernetes/how-to/deploy-application-kubernetes-helm/).

Although Kubernetes still seemed more like infrastructure we looked at Helm in this exercise which offers a more application persepctive. [Draft](../ex7) is a framework that leverages Helm (and Kubernetes) to provide a higher level of abstraction as an application.