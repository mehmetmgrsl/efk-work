# WIP

## Why EFK (Elasticsearch, Fluentd, Kibana) Stack?

- To allow users to easily and efficiently manage (collect, store and analyze) log data.

## What is EFK?

- **E**lasticsearch : Fast and scalable log storage and retrieval
- **F**luentd: Handles log collection and forwarding
- **K**ibana:  Provides a powerful visualization and dashboarding interface

## Build and push docker images of the java and node apps

- Java App -> [java-app-efk/ReadMe.MD](java-app-efk/ReadMe.MD)

- Node App -> [node-app/ReadMe.MD](node-app/ReadMe.MD)

## Create a K8S cluster (1 master 2 worker nodes)

- kind create cluster --name efk-cluster --config kind-config.yaml

## Create a secret to access docker registry

1. Set the following env variables:
  - DOCKER_REGISTRY_SERVER=docker.io
  - DOCKER_USER=<DOCKER_ID>
  - DOCKER_EMAIL=<yor_email>
  - DOCKER_PASSWORD=<your_psw>

2. Create the secret

```
kubectl create secret docker-registry myregistrysecret \
--docker-server=$DOCKER_REGISTRY_SERVER \
--docker-username=$DOCKER_USER \
--docker-password=$DOCKER_PASSWORD \
--docker-email=$DOCKER_EMAIL 
```

### Resource
1. [EFK - Nana Janashia](https://gitlab.com/nanuchi/efk-course-commands)
2. [Logging in Kubernetes with Elasticsearch, Fluentd and Kibana | Complete Course Overview - TechWorld with Nana](https://www.youtube.com/watch?v=I5c8Pfg2tys)