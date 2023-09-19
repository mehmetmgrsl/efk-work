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

- Access Kibana locally

   ```kubectl port-forward deployment/kibana-kibana 5601```

    Open a browser and access it via -> http://localhost:5601


4. Install Fluentd

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install fluentd bitnami/fluentd
```

```
kubectl get pods
NAME                                                     READY   STATUS    RESTARTS      AGE
elasticsearch-master-0                                   1/1     Running   1 (8h ago)    12h
elasticsearch-master-1                                   1/1     Running   1 (8h ago)    12h
elasticsearch-master-2                                   1/1     Running   1 (8h ago)    12h
fluentd-0                                                1/1     Running   0             2m54s
fluentd-nvp85                                            1/1     Running   4 (57s ago)   2m54s
java-app-65b6dc8dd-mx4q2                                 1/1     Running   1 (8h ago)    12h
kibana-kibana-769dfd4cdf-bp4zr                           1/1     Running   1 (24m ago)   53m
nginx-ingress-ingress-nginx-controller-759897f58-lxwf7   1/1     Running   0             42m
node-app-599d64967c-dp69z                                1/1     Running   1 (58m ago)   12h
```

5. Update the configmap of Fluentd

```kubectl edit cm fluentd-forwarder-cm```

5.1. Change ```@type stdout```  to  ```@type null``` like below to not see the healthcheck logs:

from: 
```
  fluentd-output.conf: |
    # Throw the healthcheck to the standard output instead of forwarding it
    <match fluentd.healthcheck>
      @type stdout
    </match>
```
to:

```
  fluentd-output.conf: |
    # Throw the healthcheck to the standard output instead of forwarding it
    <match fluentd.healthcheck>
      @type null
    </match>
```

5.2. Change ```path /var/log/containers/*.log```  to ```path /var/log/containers/*-app*.log``` to exclude all logs except our apps' logs (java-app-efk and node-app)

from: 

```
    <source>
      @type tail
      path /var/log/containers/*.log
```

to:

```
    <source>
      @type tail
      path /var/log/containers/*-app*.log      
```

and delete the following lines. Because we already excluded fluentd logs with the change above:

```
      # exclude Fluentd logs
      exclude_path /var/log/containers/*fluentd*.log
```
5.3. Instead of forwarding logs to aggregator, we can send it to the console to see the logs, change the following lines:

from: 

```
    # Forward all logs to the aggregators
    <match **>
      @type forward
      <server>
        host fluentd-0.fluentd-headless.default.svc.cluster.local
        port 24224
      </server>
      <buffer>
        @type file
        path /opt/bitnami/fluentd/logs/buffers/logs.buffer
        flush_thread_count 2
        flush_interval 5s
      </buffer>
    </match>
```
to:

```
    # Print all logs on the console
    <match **>
      @type stdout
    </match>
```

5.4. Restart the daemonset to use the updated config map.
```kubectl rollout restart daemonset/fluentd```

5.5. Check the log of fluentd, to see only java and node apps' logs:

```kubectl logs <name-of-the-fluentd-pod>```



### Resources
1. [EFK - Nana Janashia](https://gitlab.com/nanuchi/efk-course-commands)
2. [Logging in Kubernetes with Elasticsearch, Fluentd and Kibana | Complete Course Overview - TechWorld with Nana](https://www.youtube.com/watch?v=I5c8Pfg2tys)
3. https://gitlab.com/nanuchi/efk-course-commands/-/blob/master/commands.md?ref_type=heads