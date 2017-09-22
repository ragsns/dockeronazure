# Docker to Helm Hands-On Labs

## Exercise 2: Gentle Intro. to Docker

**STOP**! Please ensure that you have met all the prereqisites as mentioned [earlier](../../README.md).

You can safely skip this exercise which is a gentle intro if you're already reasonably familiar with Docker.

### Ensure that you are in the right sub-directory

Ensure that you are in sub-directory ex2.

```
cd <path-to-hol-folder>/dockeronazure/exercises/ex2
```

### Login to master

`ssh` into the master which we'll use as a VM with Docker pre-installed. You can pretty much do these exerises wherever Docker is installed.

#### ssh into the master

ssh into the master as below.

```
ssh <user-id-of-master>@<ip-of-master>
```

Accept the prompt and if everything goes well you should be able to be logged into master as shown below.

```
ragsns@swarmm-master-26830735-0:~$ 
```

#### Try some Docker commands

We will try some Docker commands and try to understand the power and simplicity of Docker in the process.

Start with the command

```
docker --help
```

Notice the `run`, `images`, `ps` and a few other commands that we'll execute.

Let's start with

```
docker ps
```

Which yields the following output

```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

Which indicates no Dockerized apps. are running.

The following command would initially show no Dockerized images running locally.

```
docker images
```

Lets run our `Hello World` program.

```
docker run hello-world
```

The first line of the output might look something like below.

```
Unable to find image 'hello-world:latest' locally
```

The image will be pulled from the `docker.io` public repository and run within the container. The container terminates when the app. terminates.

Run the following command which will indicate that the image `hello-world` was downloaded locally.

```
docker images
```

Let's try the following now

```
docker run -p 80:8080 -d ragsns/spring-boot
```

This pulls the spring boot app. and runs it locally forwarding the port 8080 to 80 locally. Notice that it uses a Docker Hub ID `ragsns` unlike some generally available images.

Run the following command

```
docker ps
```

The output would look something like below. Notice that the port 8080 of the application has been forwarded to 80 locally.

```
CONTAINER ID        IMAGE                COMMAND                CREATED             STATUS              PORTS                  NAMES
b471f48b2c40        ragsns/spring-boot   "/bin/sh -c /run.sh"   16 seconds ago      Up 14 seconds       0.0.0.0:80->8080/tcp   confident_carson
```

Using the `curl` command to the end point as below

```
curl localhost
```

should yield an output as below.

```
Hello World!
```

You can get the logs by running the following command

```
docker logs <name-or-id-of-container>
```

which will yield an output that looks something like below.

```

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v1.3.0.BUILD-SNAPSHOT)

2017-09-22 19:00:03.342  INFO 8 --- [           main] Example                                  : Starting Example on 664e5c98b950 with PID 8 (/myproject-0.0.1-SNAPSHOT.jar started by root in /)
2017-09-22 19:00:03.351  INFO 8 --- [           main] Example                                  : No profiles are active
2017-09-22 19:00:03.441  INFO 8 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@2c367a75: startup date [Fri Sep 22 19:00:03 UTC 2017]; root of context hierarchy
2017-09-22 19:00:04.205  INFO 8 --- [           main] o.s.b.f.s.DefaultListableBeanFactory     : Overriding bean definition for bean 'beanNameViewResolver' with a different definition: replacing [Root bean: class [null]; scope=; abstract=false; lazyInit=false; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=org.springframework.boot.autoconfigure.web.ErrorMvcAutoConfiguration$WhitelabelErrorViewConfiguration; factoryMethodName=beanNameViewResolver; initMethodName=null; destroyMethodName=(inferred); defined in class path resource [org/springframework/boot/autoconfigure/web/ErrorMvcAutoConfiguration$WhitelabelErrorViewConfiguration.class]] with [Root bean: class [null]; scope=; abstract=false; lazyInit=false; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration$WebMvcAutoConfigurationAdapter; factoryMethodName=beanNameViewResolver; initMethodName=null; destroyMethodName=(inferred); defined in class path resource [org/springframework/boot/autoconfigure/web/WebMvcAutoConfiguration$WebMvcAutoConfigurationAdapter.class]]
2017-09-22 19:00:04.865  INFO 8 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat initialized with port(s): 8080 (http)
2017-09-22 19:00:04.880  INFO 8 --- [           main] o.apache.catalina.core.StandardService   : Starting service Tomcat
2017-09-22 19:00:04.882  INFO 8 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/8.0.28
2017-09-22 19:00:04.955  INFO 8 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2017-09-22 19:00:04.956  INFO 8 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1521 ms
2017-09-22 19:00:05.217  INFO 8 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean        : Mapping servlet: 'dispatcherServlet' to [/]
2017-09-22 19:00:05.221  INFO 8 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'characterEncodingFilter' to: [/*]
2017-09-22 19:00:05.222  INFO 8 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
2017-09-22 19:00:05.222  INFO 8 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'httpPutFormContentFilter' to: [/*]
2017-09-22 19:00:05.222  INFO 8 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'requestContextFilter' to: [/*]
2017-09-22 19:00:05.413  INFO 8 --- [           main] s.w.s.m.m.a.RequestMappingHandlerAdapter : Looking for @ControllerAdvice: org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@2c367a75: startup date [Fri Sep 22 19:00:03 UTC 2017]; root of context hierarchy
2017-09-22 19:00:05.489  INFO 8 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/]}" onto java.lang.String Example.home()
2017-09-22 19:00:05.490  INFO 8 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/env]}" onto public java.lang.String Example.env(org.springframework.ui.Model)
2017-09-22 19:00:05.490  INFO 8 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/exit]}" onto java.lang.String Example.exit()
2017-09-22 19:00:05.495  INFO 8 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error],produces=[text/html]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest)
2017-09-22 19:00:05.496  INFO 8 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
2017-09-22 19:00:05.531  INFO 8 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2017-09-22 19:00:05.532  INFO 8 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2017-09-22 19:00:05.580  INFO 8 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2017-09-22 19:00:05.694  INFO 8 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2017-09-22 19:00:05.782  INFO 8 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2017-09-22 19:00:05.788  INFO 8 --- [           main] Example                                  : Started Example in 2.967 seconds (JVM running for 3.388)
```

Try another end point

```
curl localhost/env
```

which should yield an output below.

```
Environment : 
PATH = /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME = b471f48b2c40
JAVA_DEBIAN_VERSION = 8u66-b17-1~bpo8+1
CA_CERTIFICATES_JAVA_VERSION = 20140324
PWD = /
JAVA_VERSION = 8u66
LANG = C.UTF-8
HOME = /root
```

Finally, we'll try another end point as below

```
curl localhost/exit
```

This endpoint terminates the app (I should have warned you before) and you'll see the following output.

```
curl localhost/exit
curl: (52) Empty reply from server
```

Now run the following command

```
docker ps -a
```

Which yields an output which looks something like below indicating the app terminated.

```
CONTAINER ID        IMAGE                COMMAND                CREATED             STATUS                     PORTS               NAMES
2fb5ba74f7e4        ragsns/spring-boot   "/bin/sh -c /run.sh"   12 seconds ago      Exited (0) 3 seconds ago                       unruffled_carson
```

Note that Application self-healing is not supported in simple scenarios. We will look at apects of application self-healing in the context of the same app. which is provided by some of the Docker frameworks on Azure.

#### Pushing a Docker image

Let's start with the following command on the master.

```
git clone https://github.com/ragsns/hello-world-spring-boot
```

`cd` to the directory.

```
cd hello-world-spring-boot
```

Look at the `Dockerfile` that looks something like below.

```
FROM java
ADD ./target/myproject-0.0.1-SNAPSHOT.jar /myproject-0.0.1-SNAPSHOT.jar
ADD ./run.sh /run.sh
RUN chmod a+x /run.sh
EXPOSE 8080:8080
CMD /run.sh
```

It's somewhat self explanatory. It's based on a `java` container that is pulled from the public repository `docker.io` and then the `JAR` file is executed based on the `run.sh` command which is just a single line as below

```
java -jar /myproject-0.0.1-SNAPSHOT.jar
```

Essentially the Java Spring Boot App is Dockerized via the `Dockerfile`.

To build a Dockerized app, run the following command, making sure to substitute your own docker id.

```
docker build . -t <docker-id>/hello-world-spring-boot
```

You'll see an output that looks something like below indicating the image is in your local repository

```
Sending build context to Docker daemon 25.53 MB
Step 1/6 : FROM java
latest: Pulling from library/java
5040bd298390: Pull complete 
fce5728aad85: Pull complete 
76610ec20bf5: Pull complete 
60170fec2151: Pull complete 
e98f73de8f0d: Pull complete 
11f7af24ed9c: Pull complete 
49e2d6393f32: Pull complete 
bb9cdec9c7f3: Pull complete 
Digest: sha256:c1ff613e8ba25833d2e1940da0940c3824f03f802c449f3d1815a66b7f8c0e9d
Status: Downloaded newer image for java:latest
 ---> d23bdf5b1b1b
Step 2/6 : ADD ./target/myproject-0.0.1-SNAPSHOT.jar /myproject-0.0.1-SNAPSHOT.jar
 ---> 0d9a0e5b6f22
Removing intermediate container 0c2ecd0194b7
Step 3/6 : ADD ./run.sh /run.sh
 ---> e432e1b975b9
Removing intermediate container a9b159c0d7d2
Step 4/6 : RUN chmod a+x /run.sh
 ---> Running in b71514fdc758
 ---> 21bd2d0a30ec
Removing intermediate container b71514fdc758
Step 5/6 : EXPOSE 8080:8080
 ---> Running in ea5f9abe50ed
 ---> f75b1399ad9c
Removing intermediate container ea5f9abe50ed
Step 6/6 : CMD /run.sh
 ---> Running in 8dc844258f69
 ---> f05113e81a6e
Removing intermediate container 8dc844258f69
Successfully built f05113e81a6e
```

The following command will verify that the image is indeed contained in the local repository.

```
docker images
```

You can push this to the Docker Hub repository using the following command

```
docker push <docker-id>/hello-world-spring-boot
```

It's available for everyone in the world to use just like you did. You can verify that by issuing the following command

```
docker search <docker-id>
```

We merely scratched at the surface of docker commands. You can invoke `bash` on a running container, `attach` to a container and so on. Feel free to try some of the commands.


### Summary and Next Steps

We got acquainted with some simple Docker commands that are handy in dealing with it.

From an application perspective containers solve the problem of application mobility (including port forwarding) but they still don't make it easy to scale and self heal which we'll look into in subsequent exercises.

Next, we'll walk through a more involved example including ability to mount volumes based on [Docker Compose](../ex3).
