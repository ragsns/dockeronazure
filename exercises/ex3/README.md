# Docker to Helm Hands-On Labs

## Exercise 3: Docker Compose

**STOP**! Please ensure that you have met all the prereqisites as mentioned [earlier](../../README.md).

### Ensure that you are in the right sub-directory

Ensure that you are in sub-directory ex3.

```
cd <path-to-hol-folder>/dockeronazure/exercises/ex3
```

### Docker tutorial Docker Compose Example

Rarely is an application a monolith. It's often composed of different services (like an authentication service, database service, queueing service, etc.).

Typically an application is composed of services and Docker Compose is a way to deploy this application.

We will run through the example in its entirety at [https://docs.docker.com/compose/gettingstarted/](https://docs.docker.com/compose/gettingstarted/).

In this application a web component is connected to a persistent store (Redis) and it's handled in the `docker-compose.yml` file. More complex examples are available.


### Summary and Next Steps

We started with some simple Docker commands in earlier exercise and just used Docker compose.

Docker compose is a way of composing services and standing an app up but they still don't make it easy to scale and self heal which we'll look into in subsequent exercises.

Next, we'll start looking at some frameworks that support some application tenets like scaling, self-healing and so on using [Docker Swarm](../ex4).
