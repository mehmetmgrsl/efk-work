### Why EFK (Elasticsearch, FluentBit, Kibana) Stack?

- To allow users to easily and efficiently manage (collect, store and analyze) log data.

### What is EFK?

- **E**lasticsearch : Fast and scalable log storage and retrieval
- **F**luentbit: Handles log collection and forwarding
- **K**ibana:  Provides a powerful visualization and dashboarding interface

### Build and push docker images of the java and node apps

- Java App -> [java-app-efk/ReadMe.MD](java-app-efk/ReadMe.MD)

- Node App -> [node-app/ReadMe.MD](node-app/ReadMe.MD)

### Create a K8S cluster

- minikube start --memory 4096 --cpus 4 --kubernetes-version v1.25.14

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

### Deploy the java application

```kubectl apply -f java-app-efk/deployment.yaml```

```
kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
java-app-558bb79ff6-54rcm   1/1     Running   0          52s
```

- Check the logs

   ```kubectl logs java-app-558bb79ff6-54rcm```

### Deploy the nodejs application

```kubectl apply -f node-app/deployment.yaml```   

```
kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
java-app-558bb79ff6-54rcm   1/1     Running   0          7m52s
node-app-66c684f4b5-9pssm   1/1     Running   0          60s
```

- Check the logs

   ```kubectl logs node-app-66c684f4b5-9pssm```

### Install Elastic Stack (EFK - Elastic, FluentBit, Kibana) (4*)

Apply the following commands:

- ```kubectl apply -f 01-namespace.yaml```

- ```kubectl apply -f 02-secret.yaml```

- ```kubectl apply -f 03-es-svs.yaml```

- ```kubectl apply -f 04-es-sts.yaml```

- ```kubectl apply -f 05-kibana-svc.yaml```

- ```kubectl apply -f 06-kibana-deploy.yaml```

- ```kubectl apply -f 07-fluentbit-sa.yaml```

- ```kubectl apply -f 08-fluentbit-role.yaml```

- ```kubectl apply -f 09-fluentbit-rb.yaml```

- ```kubectl apply -f 10-fluentbit-cm.yaml```

- ```kubectl apply -f 11-fluentbit-ds.yaml```


1. Make sure Elasticsearch cluster is working:

- ```kubectl port-forward svc/elasticsearch 9200 -n kube-logging```

- ```curl http://localhost:9200/_cluster/health/?pretty -u elastic```
  - psw: password

2. Verify Kibana Deployment 

```kubectl port-forward <kibana-pod-name> 5601:5601  -n kube-logging```

i.e.

```kubectl port-forward  kibana-77c8bb6f94-f46zf 5601:5601 -n kube-logging```

visit http://localhost:5601/ (user/psw -> elastic/password)


### Resources
1* [EFK - Nana Janashia](https://gitlab.com/nanuchi/efk-course-commands)

2* [Logging in Kubernetes with Elasticsearch, Fluentd and Kibana | Complete Course Overview - TechWorld with Nana](https://www.youtube.com/watch?v=I5c8Pfg2tys)

3* https://gitlab.com/nanuchi/efk-course-commands/-/blob/master/commands.md?ref_type=heads

4* [Protected EFK Stack Setup for Kubernetes - Priyanshi Chauhan](https://blog.opstree.com/2023/01/24/protected-efk-stack-setup-for-kubernetes/)