# Docker to Helm Hands-On Labs

## Exercise 9: A Complete Multi-Container Application

**STOP**! Please ensure that you have met all the prereqisites as mentioned [earlier](../../README.md) and you have access to a Kubernetes Cluster as outlined in [exercise 5a] (../ex5a/README.md) ***or*** [exercise 5b] (../ex5b/README.md).

### Set context to the previously created Kubernetes Cluster

Verify that you're logged into Azure and set the right subscription with the following command as mentioned in a [previous execrise](../ex1/README.md).

```
az account show
```

Set the context to the previously created cluster with the following command. If using a cluster in another cloud provider (such as Google cloud or AWS) substitute the name with the name of the Kubernetes cluster.

```
kubectl config use-context $UNIQUE_NAME
```

### The Application

Clone the repository with the following command.

```
git clone https://github.com/ragsns/gowebapp
```

`cd` into the sub-directory

```
cd gowebapp
```

It's a `go` application that uses MySQL as the backing store.

### Create the RBAC workaround

```
kubectl create serviceaccount --namespace kube-system tiller
```

```
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
```




### Install MySQL

Install ```MySQL``` with the following command. You certainly want to change the password in a production scenario.

```
helm install --name mysql --set mysqlRootPassword=changeme stable/mysql
```

You should see an output that looks something like below confirmation that ```MySQL``` installed successfully.

```
NAME:   mysql
LAST DEPLOYED: Sun Mar  4 19:09:47 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/PersistentVolumeClaim
NAME         STATUS   VOLUME   CAPACITY  ACCESSMODES  STORAGECLASS  AGE
mysql-mysql  Pending  default  1s

==> v1/Service
NAME         CLUSTER-IP    EXTERNAL-IP  PORT(S)   AGE
mysql-mysql  10.0.204.203  <none>       3306/TCP  1s

==> v1beta1/Deployment
NAME         DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
mysql-mysql  1        1        1           0          1s

==> v1/Secret
NAME         TYPE    DATA  AGE
mysql-mysql  Opaque  2     1s


NOTES:
MySQL can be accessed via port 3306 on the following DNS name from within your cluster:
mysql-mysql.default.svc.cluster.local

To get your root password run:

    kubectl get secret --namespace default mysql-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo

To connect to your database:

1. Run an Ubuntu pod that you can use as a client:

    kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il

2. Install the mysql client:

    $ apt-get update && apt-get install mysql-client -y

3. Connect using the mysql cli, then provide your password:
    $ mysql -h mysql-mysql -p
```

#### Update the MySQL schema

We'll follow the steps outlined in the output with some modifications as below. Let's start by running an ubuntu pod as below.

```
kubectl run -i --tty ubuntu --image=ubuntu:16.04 -- bash -il
```

Hit <ctrl-d> or type the following to exit out of the shell.

```
exit
```

Copy the schema file `mysql.sql` to the container with the following command

```
kubectl cp config/mysql.sql $(kubectl get pods | grep ubuntu | awk '{print $1}'):.
```

Attach again to the container with the following command

```
kubectl attach $(kubectl get pods | grep ubuntu | awk '{print $1}') -i -t
```

Once you get the ```root``` command prompt, install the MySQL client inside the ubuntu container as below.

```
apt-get update && apt-get install mysql-client -y
```

Now install the schema in the database from inside the ubuntu container by connecting to the MySQL database as below.

```
mysql -h mysql-mysql -pchangeme < mysql.sql
```

If everything works well you can just ignore the warning as below.

```
mysql: [Warning] Using a password on the command line interface can be insecure.
```
Let's verify the schemas have been created properly using the `mysql` client as below.

```
mysql -h mysql-mysql -pchangeme
```

After you get the mysql prompt type the following commands

```
use gowebapp;
show tables;
```

You should see an output that looks something like below.

```
+--------------------+
| Tables_in_gowebapp |
+--------------------+
| note               |
| user               |
| user_status        |
+--------------------+
3 rows in set (0.00 sec)
```

Exit the `mysql` client with the following command or hitting <ctrl-d>.

```
exit
```

Exit the container with the following command or hitting <ctrl-d>.

```
exit
```

and clean up the container with the following command.

```
kubectl delete deployment/ubuntu
```

### Dockerize the application

Let's create an image as below, making sure you substitute your Docker ID.

```
docker build -t <your-docker-id>/gowebapp .
```

The output should look something like below.

```
Sending build context to Docker daemon  1.174MB
Step 1/8 : FROM golang:1.8
 ---> 0d283eb41a92
Step 2/8 : WORKDIR /go/src/app
 ---> Using cache
 ---> 709af0642847
Step 3/8 : COPY . .
 ---> 2050d7f6b04a
Step 4/8 : RUN go get -d -v ./...
 ---> Running in d082ed508357
github.com/boltdb/bolt (download)
github.com/go-sql-driver/mysql (download)
github.com/jmoiron/sqlx (download)
Fetching https://gopkg.in/mgo.v2?go-get=1
Parsing meta tags from https://gopkg.in/mgo.v2?go-get=1 (status code 200)
get "gopkg.in/mgo.v2": found meta tag main.metaImport{Prefix:"gopkg.in/mgo.v2", VCS:"git", RepoRoot:"https://gopkg.in/mgo.v2"} at https://gopkg.in/mgo.v2?go-get=1
gopkg.in/mgo.v2 (download)
Fetching https://gopkg.in/mgo.v2/bson?go-get=1
Parsing meta tags from https://gopkg.in/mgo.v2/bson?go-get=1 (status code 200)
get "gopkg.in/mgo.v2/bson": found meta tag main.metaImport{Prefix:"gopkg.in/mgo.v2", VCS:"git", RepoRoot:"https://gopkg.in/mgo.v2"} at https://gopkg.in/mgo.v2/bson?go-get=1
get "gopkg.in/mgo.v2/bson": verifying non-authoritative meta tag
Fetching https://gopkg.in/mgo.v2?go-get=1
Parsing meta tags from https://gopkg.in/mgo.v2?go-get=1 (status code 200)
Fetching https://golang.org/x/crypto/bcrypt?go-get=1
Parsing meta tags from https://golang.org/x/crypto/bcrypt?go-get=1 (status code 200)
get "golang.org/x/crypto/bcrypt": found meta tag main.metaImport{Prefix:"golang.org/x/crypto", VCS:"git", RepoRoot:"https://go.googlesource.com/crypto"} at https://golang.org/x/crypto/bcrypt?go-get=1
get "golang.org/x/crypto/bcrypt": verifying non-authoritative meta tag
Fetching https://golang.org/x/crypto?go-get=1
Parsing meta tags from https://golang.org/x/crypto?go-get=1 (status code 200)
golang.org/x/crypto (download)
github.com/haisum/recaptcha (download)
github.com/gorilla/sessions (download)
github.com/gorilla/context (download)
github.com/gorilla/securecookie (download)
github.com/josephspurrier/csrfbanana (download)
github.com/julienschmidt/httprouter (download)
github.com/justinas/alice (download)
Removing intermediate container d082ed508357
 ---> 946c5c4b52a1
Step 5/8 : RUN go install -v ./...
 ---> Running in 23166b91edfc
github.com/boltdb/bolt
github.com/go-sql-driver/mysql
gopkg.in/mgo.v2/internal/json
github.com/jmoiron/sqlx/reflectx
github.com/jmoiron/sqlx
gopkg.in/mgo.v2/internal/scram
golang.org/x/crypto/blowfish
golang.org/x/crypto/bcrypt
github.com/haisum/recaptcha
gopkg.in/mgo.v2/bson
app/vendor/app/shared/passhash
app/vendor/app/shared/recaptcha
github.com/gorilla/context
github.com/gorilla/securecookie
github.com/julienschmidt/httprouter
app/vendor/app/route/middleware/logrequest
github.com/justinas/alice
github.com/gorilla/sessions
app/vendor/app/shared/email
app/vendor/app/route/middleware/httprouterwrapper
gopkg.in/mgo.v2
app/vendor/app/shared/session
github.com/josephspurrier/csrfbanana
app/vendor/app/shared/view
app/vendor/app/route/middleware/acl
app/vendor/app/route/middleware/pprofhandler
app/vendor/app/shared/jsonconfig
app/vendor/app/shared/server
app/vendor/app/shared/view/plugin
app/vendor/app/shared/database
app/vendor/app/model
app/vendor/app/controller
app/vendor/app/route
app
Removing intermediate container 23166b91edfc
 ---> 9309fae5ca2f
Step 6/8 : ENV PORT 8080
 ---> Running in dfe3b0645f2b
Removing intermediate container dfe3b0645f2b
 ---> 0a006d4d310a
Step 7/8 : EXPOSE 8080
 ---> Running in 9cd305e68ab9
Removing intermediate container 9cd305e68ab9
 ---> 475354a28c2d
Step 8/8 : CMD ["app"]
 ---> Running in 480089e4aff8
Removing intermediate container 480089e4aff8
 ---> aa38064c88d8
Successfully built aa38064c88d8
Successfully tagged ragsns/gowebapp:latest
```

Let's push this to the public Docker repository with the following command.

```
docker push <your-docker-id>/gowebapp
```

We'll deploy a pod front ended by a load balancer as below. Substibtute `ragsns` to `<your-docker-id>` in the `gowebapp.yaml` file to use the Docker image you just pushed.

```
kubectl create -f gowebapp.yaml
```

Let's watch the external IP creation with the following command

```
kubectl get service gowebapp --watch
```

You should see an output that looks something like below.

```              
NAME       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
gowebapp   10.0.82.133   <pending>     80:31559/TCP   19s
gowebapp   10.0.82.133   52.170.192.130   80:31559/TCP   4m
^C
```

Once the external IP (`52.170.192.130`) is available, you can <ctrl-c> and access the application via that IP.
 
### Cleanup

Issue the following command to create the artifacts we created in this exercise.

```
kubectl delete deploy/gowebapp
kubectl delete service/gowebapp
```

### Summary and Next Steps

We started with some simple Docker commands.

In this exercise we installed a simple web application that persists data in a `mysql` database.

Congrats, you are now an Azure Kubernetes Hero!