# Autoscaling Kafka Consumer Application using KEDA

This guide describes how can be Kafka Consumer application autoscaled by KEDA on Kubernetes. The application is being scaled based on lag in the Kafka topic. If there isn't any traffic the application is autoscaled to 0 replicas, if there's some load the number of replicas is being scaled up until the number of partitions.

This resources have been tested Azure Environment using Azure Event Hubs as Apache Kafka provider, but it could be modified to connect any existing apache Kafka.

This demo leverage KEDA Apache Kafka scaler, you can see the documentation here: https://keda.sh/docs/2.12/scalers/apache-kafka/

## 0. Provisioning Azure Event Hubs within a consumer group and topic

## 1. Install KEDA

Follow the steps in KEDA Documentation for installing KEDA using Helm in any Kubernetes cluster: https://keda.sh/docs/2.12/deploy/#helm 

## 2. Create namespace for the demo
Create "keda-demo" namespace: 

 ```bash
kubectl create ns keda-demo
 ```

## 2. Deploy Kafka Consumer application

Create the secret with the base64 credentials for connecting Kafka/Azure Event Hubs:

 ```bash
kubectl apply -f deploy/1-secret.yaml
 ```

Deploy Kafka Consumer application with the following command:

 ```bash
kubectl apply -f deploy/2-kafka-consumer.yaml
 ```

Check the consumer has been able to connect to Kafka (Azure Event Hubs):
 ```bash
kubectl -n keda-demo logs deployment.apps/kafka-consumer
 ```

You should see similar output logs:
 ```bash

 ```

## 3. 

## Crear recursos
En la carpeta **keda/deploy** crear:
kubeclt apply -f

### Generar eventos
Requisitos instalar go

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
kubectl get --raw "/apis/external.metrics.k8s.io/v1beta1/namespaces/keda-demo/s0-kafka-keda?labelSelector=scaledobject.keda.sh%2Fname%3Dkafka-scaledobject"
```