# Autoscaling Kafka Consumer Application using KEDA

This guide describes how can be Kafka Consumer application autoscaled by KEDA on Kubernetes. The application is being scaled based on lag in the Kafka topic. If there isn't any traffic the application is autoscaled to 0 replicas, if there's some load the number of replicas is being scaled up until the number of partitions.

This resources have been tested Azure Environment using Azure Event Hubs as Apache Kafka provider, but it could be modified to connect any existing Apache Kafka.

This demo leverage KEDA Apache Kafka scaler, you can see the documentation here: https://keda.sh/docs/2.12/scalers/apache-kafka/

### Diagram:
![Diagram](images/diagram.png?raw=true "Autoscaling of Kafka Consumer application")

## 0. Provisioning Azure Event Hubs within a consumer group and topic

## 1. Install KEDA

Follow the steps in KEDA Documentation for installing KEDA using Helm (https://helm.sh/) in any Kubernetes cluster: https://keda.sh/docs/2.12/deploy/#helm 

## 2. Create namespace for the demo
Create "keda-demo" namespace: 

 ```bash
kubectl create ns keda-demo
 ```

## 3. Deploy Kafka Consumer application

Create the secret with the base64 credentials for connecting Kafka/Azure Event Hubs:

 ```bash
kubectl apply -f deploy/1-secret.yaml
 ```

Deploy Kafka Consumer application with the following command:

 ```bash
kubectl apply -f deploy/2-kafka-consumer.yaml
 ```
Check the status of the Deployment kafka-consumer:

```bash
kubectl -n keda-demo get deployment.apps/kafka-consumer
 ```

You should see similar output
```bash
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
kafka-consumer   1/1     1            1           6m41s
```

Check the consumer has been able to connect to Kafka (Azure Event Hubs):
 ```bash
kubectl -n keda-demo logs deployment.apps/kafka-consumer
 ```
You should see similar output logs:
 ```bash
2023/12/15 22:49:11 KAFKA_EVENTHUB_ENDPOINT - kedans.servicebus.windows.net:9093
2023/12/15 22:49:11 KAFKA_EVENTHUB_CONSUMER_GROUP - group_keda
2023/12/15 22:49:11 KAFKA_EVENTHUB_TOPIC - keda
%7|1702680551.062|INIT|rdkafka#consumer-1| [thrd:app]: librdkafka v1.4.0 (0x10400ff) rdkafka#consumer-1 initialized (builtin.features gzip,snappy,ssl,sasl,regex,lz4,sasl_plain,sasl_scram,plugins,zstd,sasl_oauthbearer, STATIC_LINKING CC GXX PKGCONFIG INSTALL GNULD LDS LIBDL PLUGINS ZLIB SSL ZSTD HDRHISTOGRAM SYSLOG SNAPPY SOCKEM SASL_SCRAM SASL_OAUTHBEARER CRC32C_HW, debug 0x2000)
%7|1702680551.062|SUBSCRIBE|rdkafka#consumer-1| [thrd:main]: Group "group_keda": subscribe to new subscription of 1 topics (join state init)
%7|1702680551.062|REBALANCE|rdkafka#consumer-1| [thrd:main]: Group "group_keda" is rebalancing in state init (join-state init) without assignment: unsubscribe
2023/12/15 22:49:11 press ctrl+c to exit
2023/12/15 22:49:11 waiting for messages...
%7|1702680552.338|JOIN|rdkafka#consumer-1| [thrd:main]: Group "group_keda": postponing join until up-to-date metadata is available
%7|1702680552.343|REJOIN|rdkafka#consumer-1| [thrd:main]: Group "group_keda": subscription updated from metadata change: rejoining group
%7|1702680552.343|REBALANCE|rdkafka#consumer-1| [thrd:main]: Group "group_keda" is rebalancing in state up (join-state init) without assignment: group rejoin
%7|1702680554.062|JOIN|rdkafka#consumer-1| [thrd:main]: sasl_ssl://kedans.servicebus.windows.net:9093/0: Joining group "group_keda" with 1 subscribed topic(s)
%7|1702680557.016|ASSIGNOR|rdkafka#consumer-1| [thrd:main]: Group "group_keda": "range" assignor run for 1 member(s)
%7|1702680557.044|ASSIGN|rdkafka#consumer-1| [thrd:main]: Group "group_keda": new assignment of 20 partition(s) in join state wait-assign-rebalance_cb
%7|1702680557.044|OFFSET|rdkafka#consumer-1| [thrd:main]: GroupCoordinator/0: Fetch committed offsets for 20/20 partition(s)
%7|1702680557.078|FETCH|rdkafka#consumer-1| [thrd:main]: Partition keda [0] start fetching at offset 386
%7|1702680557.078|FETCH|rdkafka#consumer-1| [thrd:main]: Partition keda [1] start fetching at offset 320
%7|1702680557.079|FETCH|rdkafka#consumer-1| [thrd:main]: Partition keda [2] start fetching at offset 265
%7|1702680557.079|FETCH|rdkafka#consumer-1| [thrd:main]: Partition keda [3] start fetching at offset 257
%7|1702680557.079|FETCH|rdkafka#consumer-1| [thrd:main]: Partition keda [4] start fetching at offset 327
...
 ```

## 4. Deploy ScaleObject for enabling Autoscaling with KEDA for Kafka Consumer application

First, we have to create Trigger Authentication object needed for Scaled Object
```bash
kubectl apply -f deploy/3-trigger-auth.yaml
```
<sub>Note: This Trigger Authentication object uses the Secret created in the Step 3.</sub>

Create the ScaledObject with the following command:
```bash
kubectl apply -f deploy/4-kafka-consumer-scaledobject.yaml
```

Check that KEDA has been able to access metrics and is correctly defined for autoscaling:
```bash
kubectl -n keda-demo get scaledobject kafka-consumer-scaledobject
```
You should see similar output, `READY` should be `True`:
```bash
NAME                          SCALETARGETKIND      SCALETARGETNAME   MIN   MAX   TRIGGERS   AUTHENTICATION               READY   ACTIVE   FALLBACK   PAUSED    AGE
kafka-consumer-scaledobject   apps/v1.Deployment   kafka-consumer                kafka      eventhub-kafka-triggerauth   True    True     False      Unknown   66s
```
Since there aren't any messages in the Kafka topic, the Kafka Consumer application should be autoscaled to zero, run the following command:
```bash
kubectl -n keda-demo get deployments.apps/kafka-consumer
```
You should see a similar output, Deployment `kafka-consumer` has been autoscaled to 0 replicas:
```bash
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
kafka-consumer   0/0     0            0           44s
```

## 5. Run Kafka producer 

Note: You need to install Go runtime. Look Golang documentation.

Go to the "producer" directory:

En la carpeta **keda/producer**
- Exportar las variables del archivo *envkeda*
- Ejecutar: go run producer.go

```bash
DESA:(aksveu2pocd10)-s89684@Azure:/mnt/d/demo/keda/producer$ go run producer.go
Event Hubs broker [kedans.servicebus.windows.net:9093]
Event Hubs topic keda
Waiting for ctrl+c
sent message {"time":"Fri Nov 17 19:13:52 2023"} to partition 11 offset 49
sent message {"time":"Fri Nov 17 19:13:53 2023"} to partition 14 offset 87
```

#### Resultado

```bash
DESA:(aksveu2pocd10)-s89684@Azure:/mnt/d/demo/keda/deploy$ k -n keda get hpa
NAME                          REFERENCE                   TARGETS        MINPODS   MAXPODS   REPLICAS   AGE
keda-hpa-kafka-scaledobject   Deployment/kafka-consumer   250/50 (avg)   1         100       4          31s
DESA:(aksveu2pocd10)-s89684@Azure:/mnt/d/demo/keda/deploy$ k -n keda get po
NAME                                               READY   STATUS              RESTARTS   AGE
kafka-consumer-84cf6bf686-4n5x6                    0/1     ContainerCreating   0          26s
kafka-consumer-84cf6bf686-5787v                    0/1     ContainerCreating   0          10s
kafka-consumer-84cf6bf686-5klnj                    1/1     Running             0          11s
kafka-consumer-84cf6bf686-cgfnh                    1/1     Running             0          26s
kafka-consumer-84cf6bf686-jmwv8                    1/1     Running             0          20m
kafka-consumer-84cf6bf686-qfs4z                    1/1     Running             0          10s
kafka-consumer-84cf6bf686-qqm7d                    0/1     ContainerCreating   0          26s
kafka-consumer-84cf6bf686-vgqdh                    0/1     ContainerCreating   0          10s
keda-admission-webhooks-7cb5c79bd-wlg2s            1/1     Running             0          3d1h
keda-operator-74cd8b977c-s687j                     1/1     Running             0          94m
keda-operator-metrics-apiserver-5687545858-p5dr7   1/1     Running             0          95m
DESA:(aksveu2pocd10)-s89684@Azure:/mnt/d/demo/keda/deploy$ k -n keda get so
NAME                 SCALETARGETKIND      SCALETARGETNAME   MIN   MAX   TRIGGERS   AUTHENTICATION               READY   ACTIVE   FALLBACK   PAUSED    AGE
kafka-scaledobject   apps/v1.Deployment   kafka-consumer                kafka      eventhub-kafka-triggerauth   True    True     False      Unknown   50s
```

### REFERENCIAS
https://github.com/abhirockzz/keda-eventhubs-kafka

https://dev.to/azure/how-to-auto-scale-kafka-applications-on-kubernetes-with-keda-1k9n


Check 
```bash
kubectl get --raw "/apis/external.metrics.k8s.io/v1beta1/namespaces/keda-demo/s0-kafka-keda?labelSelector=scaledobject.keda.sh%2Fname%3Dkafka-consumer-scaledobject"
```