# Autoscaling KEDA - Kafka / Event Hubs

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