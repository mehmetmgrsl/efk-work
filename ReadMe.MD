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
helm repo add elastic https://helm.elastic.co
helm install elasticsearch elastic/elasticsearch -f elastic-custom-values.yaml
```

```
kubectl get all | grep elastic
pod/elasticsearch-master-0      1/1     Running   0          20m
pod/elasticsearch-master-1      1/1     Running   0          20m
pod/elasticsearch-master-2      1/1     Running   0          20m
service/elasticsearch-master            ClusterIP   10.104.82.89   <none>        9200/TCP,9300/TCP   20m
service/elasticsearch-master-headless   ClusterIP   None           <none>        9200/TCP,9300/TCP   20m
statefulset.apps/elasticsearch-master   3/3     20m
```
  
2. Install kibana helm chart  

```helm install kibana elastic/kibana```

```
kubectl get pods
NAME                             READY   STATUS    RESTARTS        AGE
elasticsearch-master-0           1/1     Running   1 (7h44m ago)   11h
elasticsearch-master-1           1/1     Running   1 (7h44m ago)   11h
elasticsearch-master-2           1/1     Running   1 (7h44m ago)   11h
java-app-65b6dc8dd-mx4q2         1/1     Running   1 (7h44m ago)   11h
kibana-kibana-769dfd4cdf-bp4zr   1/1     Running   0               64s
node-app-599d64967c-dp69z        1/1     Running   1 (5m13s ago)   11h
```

3. Install NGINX Ingress Controller

```
helm repo add stable https://charts.helm.sh/stable 
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install nginx-ingress ingress-nginx/ingress-nginx
```

```
kubectl get pods
NAME                                                     READY   STATUS    RESTARTS        AGE
elasticsearch-master-0                                   1/1     Running   1 (7h56m ago)   11h
elasticsearch-master-1                                   1/1     Running   1 (7h55m ago)   11h
elasticsearch-master-2                                   1/1     Running   1 (7h56m ago)   11h
java-app-65b6dc8dd-mx4q2                                 1/1     Running   1 (7h56m ago)   11h
kibana-kibana-769dfd4cdf-bp4zr                           1/1     Running   0               12m
nginx-ingress-ingress-nginx-controller-759897f58-lxwf7   1/1     Running   0               91s
node-app-599d64967c-dp69z                                1/1     Running   1 (16m ago)     11h
```


### Resources
1. [EFK - Nana Janashia](https://gitlab.com/nanuchi/efk-course-commands)
2. [Logging in Kubernetes with Elasticsearch, Fluentd and Kibana | Complete Course Overview - TechWorld with Nana](https://www.youtube.com/watch?v=I5c8Pfg2tys)
3. https://gitlab.com/nanuchi/efk-course-commands/-/blob/master/commands.md?ref_type=heads