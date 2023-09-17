# WIP

### Why EFK (Elasticsearch, Fluentd, Kibana) Stack?

- To allow users to easily and efficiently manage (collect, store and analyze) log data.

### What is EFK?

- **E**lasticsearch : Fast and scalable log storage and retrieval
- **F**luentd: Handles log collection and forwarding
- **K**ibana:  Provides a powerful visualization and dashboarding interface

### Build and push docker images of the java and node apps

- Java App -> [java-app-efk/ReadMe.MD](java-app-efk/ReadMe.MD)

- Node App -> [node-app/ReadMe.MD](node-app/ReadMe.MD)

### Create a K8S cluster

- minikube start --cpus 4 --memory 8192


### Create a secret to access docker registry
Deploy the java application
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

## Deploy the java application

```kubectl apply -f java-app-efk/deployment.yaml```

```
kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
java-app-558bb79ff6-54rcm   1/1     Running   0          52s
```

- Check the logs

   ```kubectl logs java-app-558bb79ff6-54rcm```

## Deploy the nodejs application

```kubectl apply -f node-app/deployment.yaml```   

```
kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
java-app-558bb79ff6-54rcm   1/1     Running   0          7m52s
node-app-66c684f4b5-9pssm   1/1     Running   0          60s
```

- Check the logs

   ```kubectl logs node-app-66c684f4b5-9pssm```

## Install Elastic Stack (EFK - Elastic, FluentD, Kibana)

1. Install elastic search helm chart

```
helm repo add elastic https://Helm.elastic.co
helm install elasticsearch elastic/elasticsearch
```

  

### Resource
1. [EFK - Nana Janashia](https://gitlab.com/nanuchi/efk-course-commands)
2. [Logging in Kubernetes with Elasticsearch, Fluentd and Kibana | Complete Course Overview - TechWorld with Nana](https://www.youtube.com/watch?v=I5c8Pfg2tys)
3. https://gitlab.com/nanuchi/efk-course-commands/-/blob/master/commands.md?ref_type=heads