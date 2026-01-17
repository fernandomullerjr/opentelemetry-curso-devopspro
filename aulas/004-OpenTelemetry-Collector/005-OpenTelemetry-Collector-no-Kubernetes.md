
## OpenTelemetry no Kubernetes


### Como subir no k8s?

OBS:
o service do app-a tem o "type: LoadBalancer", então dependendo do lab é necessário ajustar aqui.

- Comando para aplicar os manifestos no Kubernetes:

~~~~bash
cd /home/fernando/cursos/opentelemetry/opentelemetry-curso-devopspro/app-telemetria

kubectl apply -f k8s/ -R
~~~~


- O Otel-Collector é via Helm
https://artifacthub.io/



### Provisionando o ambiente no k8s

>
> kubectl get pods -A
NAMESPACE     NAME                               READY   STATUS              RESTARTS        AGE
default       app-a-74f7747c78-7h8nv             1/1     Running             0               18s
default       app-b-6bd9d84f56-gzvhp             1/1     Running             0               18s
default       app-c-776f947589-w2dh6             1/1     Running             0               18s
default       grafana-bfb9674f7-hjpkx            0/1     ContainerCreating   0               18s
default       jaeger-766cc89bdf-wbgjk            0/1     ContainerCreating   0               18s
default       loki-c8d9f66d4-wwhwr               0/1     ContainerCreating   0               18s
default       prometheus-68fd4f969-jswpg         0/1     ContainerCreating   0               17s
kube-system   coredns-66bc5c9577-rmfg7           1/1     Running             0               4m47s
kube-system   etcd-minikube                      1/1     Running             0               4m52s
kube-system   kube-apiserver-minikube            1/1     Running             0               4m52s
kube-system   kube-controller-manager-minikube   1/1     Running             0               4m52s
kube-system   kube-proxy-ljs8m                   1/1     Running             0               4m47s
kube-system   kube-scheduler-minikube            1/1     Running             0               4m52s
kube-system   storage-provisioner                1/1     Running             1 (4m25s ago)   4m51s
> date
Sun Jan 11 13:49:05 -03 2026




https://artifacthub.io/

https://artifacthub.io/packages/helm/opentelemetry-helm/opentelemetry-collector


Add OpenTelemetry Helm repository:

helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts

To install the chart with the release name my-opentelemetry-collector, run the following command:

helm install my-opentelemetry-collector open-telemetry/opentelemetry-collector --set mode=<value> --set image.repository="ghcr.io/open-telemetry/opentelemetry-collector-releases/opentelemetry-collector-k8s" --set command.name="otelcol-k8s"

Where the mode value needs to be set to one of daemonset, deployment or statefulset.




helm install my-opentelemetry-collector open-telemetry/opentelemetry-collector --set mode=deployment --set image.repository="ghcr.io/open-telemetry/opentelemetry-collector-releases/opentelemetry-collector-k8s" --set command.name="otelcol-k8s"


### Comando Helm para subir OTEL COLLECTOR
~~~~bash
cd /home/fernando/cursos/opentelemetry/opentelemetry-curso-devopspro/app-telemetria

helm install otel-collector open-telemetry/opentelemetry-collector --values k8s/values.yaml

~~~~




- Subindo OTEL COLLECTOR

~~~~BASH

>
> helm install otel-collector open-telemetry/opentelemetry-collector --values k8s/values.yaml
NAME: otel-collector
LAST DEPLOYED: Sun Jan 11 14:32:32 2026
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:

> date
Sun Jan 11 14:32:34 -03 2026
> helm ls -A
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
otel-collector  default         1               2026-01-11 14:32:32.059561478 -0300 -03 deployed        opentelemetry-collector-0.143.0 0.143.0
>
>
> kubectl get all
NAME                                                         READY   STATUS              RESTARTS   AGE
pod/app-a-74f7747c78-jjv2l                                   1/1     Running             0          28m
pod/app-b-6bd9d84f56-zczkv                                   1/1     Running             0          28m
pod/app-c-776f947589-9mb2h                                   1/1     Running             0          28m
pod/grafana-bfb9674f7-ms624                                  1/1     Running             0          28m
pod/jaeger-766cc89bdf-gjcsm                                  1/1     Running             0          28m
pod/loki-c8d9f66d4-lhbdx                                     1/1     Running             0          28m
pod/otel-collector-opentelemetry-collector-55d957b65-65k9w   0/1     ContainerCreating   0          11s
pod/prometheus-68fd4f969-d6gqt                               1/1     Running             0          28m

NAME                                             TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                      AGE
service/app-a                                    LoadBalancer   10.104.30.166    127.0.0.1     80:30001/TCP                                                                 28m
service/app-b                                    ClusterIP      10.105.185.75    <none>        8000/TCP                                                                     28m
service/app-c                                    ClusterIP      10.99.210.26     <none>        8000/TCP                                                                     28m
service/grafana                                  LoadBalancer   10.110.219.97    127.0.0.1     3000:30000/TCP                                                               28m
service/jaeger                                   ClusterIP      10.102.42.194    <none>        16686/TCP,14250/TCP,14268/TCP,6831/UDP,6832/UDP,9411/TCP,4317/TCP,4318/TCP   28m
service/kubernetes                               ClusterIP      10.96.0.1        <none>        443/TCP                                                                      48m
service/loki                                     ClusterIP      10.97.225.44     <none>        3100/TCP                                                                     28m
service/otel-collector-opentelemetry-collector   ClusterIP      10.101.27.82     <none>        8889/TCP,4317/TCP,4318/TCP                                                   11s
service/prometheus                               NodePort       10.111.193.185   <none>        9090:30002/TCP                                                               28m

NAME                                                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/app-a                                    1/1     1            1           28m
deployment.apps/app-b                                    1/1     1            1           28m
deployment.apps/app-c                                    1/1     1            1           28m
deployment.apps/grafana                                  1/1     1            1           28m
deployment.apps/jaeger                                   1/1     1            1           28m
deployment.apps/loki                                     1/1     1            1           28m
deployment.apps/otel-collector-opentelemetry-collector   0/1     1            0           11s
deployment.apps/prometheus                               1/1     1            1           28m

NAME                                                               DESIRED   CURRENT   READY   AGE
replicaset.apps/app-a-74f7747c78                                   1         1         1       28m
replicaset.apps/app-b-6bd9d84f56                                   1         1         1       28m
replicaset.apps/app-c-776f947589                                   1         1         1       28m
replicaset.apps/grafana-bfb9674f7                                  1         1         1       28m
replicaset.apps/jaeger-766cc89bdf                                  1         1         1       28m
replicaset.apps/loki-c8d9f66d4                                     1         1         1       28m
replicaset.apps/otel-collector-opentelemetry-collector-55d957b65   1         1         0       11s
replicaset.apps/prometheus-68fd4f969                               1         1         1       28m
> date
Sun Jan 11 14:32:47 -03 2026
~~~~



## PENDENTE
- Revisar porque Jaeger não tem dados dos Service app-a, etc. Loki tá ok trazendo logs.
- Avaliar se NAT do Minikube é o problema.
- Revisar como ficariam todos os mapeamentos de ip, service, etc, via Minikube.



## Dia 17/01/2026

- Revisar porque Jaeger não tem dados dos Service app-a, etc. Loki tá ok trazendo logs.
- Avaliar se NAT do Minikube é o problema.
- Revisar como ficariam todos os mapeamentos de ip, service, etc, via Minikube.


minikube service app-a --url

minikube service jaeger --url

apps se alcançam.

porém no grafana sigo sem conseguir ver dados dos serviços no jaeger conectado ao grafana.
na ui do jaeger só tem o service jaeger-all-in-one.

o que pode ser?


root@app-a-74f7747c78-jjv2l:/app# 
root@app-a-74f7747c78-jjv2l:/app# curl http://localhost:8000
{"message":"Esse é o serviço app-a"}root@app-a-74f7747c78-jjv2l:/app# 
root@app-a-74f7747c78-jjv2l:/app# 
root@app-a-74f7747c78-jjv2l:/app# 
root@app-a-74f7747c78-jjv2l:/app# 
root@app-a-74f7747c78-jjv2l:/app# cat | curl http://localhost:8000
{"message":"Esse é o serviço app-a"}
root@app-a-74f7747c78-jjv2l:/app# 
root@app-a-74f7747c78-jjv2l:/app# 
root@app-a-74f7747c78-jjv2l:/app# 
root@app-a-74f7747c78-jjv2l:/app# cat | curl http://app-b:8000
{"message":"Esse é o serviço app-b"}
root@app-a-74f7747c78-jjv2l:/app# 
root@app-a-74f7747c78-jjv2l:/app# 
root@app-a-74f7747c78-jjv2l:/app# ^C



Encontrei o problema! As **portas do Jaeger estão invertidas** no service. Olhe:

```yaml
# No deploy-jaeger.yaml está ERRADO:
  - name: http
    port: 4317
    targetPort: 4317  # ❌ 4317 é gRPC, não HTTP
  - name: grpc
    port: 4318
    targetPort: 4318  # ❌ 4318 é HTTP, não gRPC
```

## Correção necessária:

````yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jaeger
spec:
  selector:
    matchLabels:
      app: jaeger
  template:
    metadata:
      labels:
        app: jaeger
    spec:
      containers:
      - name: jaeger
        image: jaegertracing/all-in-one:latest
        ports:
        - containerPort: 16686
        - containerPort: 14250
        - containerPort: 14268
        - containerPort: 6831
          protocol: UDP
        - containerPort: 6832
          protocol: UDP
        - containerPort: 9411
        - containerPort: 4317  # OTLP gRPC
        - containerPort: 4318  # OTLP HTTP
        env:
        - name: COLLECTOR_ZIPKIN_HOST_PORT
          value: ":9411"
        - name: COLLECTOR_OTLP_ENABLED  # ✅ IMPORTANTE: Habilitar OTLP
          value: "true"
---
apiVersion: v1
kind: Service
metadata:
  name: jaeger
spec:
  selector:
    app: jaeger
  ports:
  - name: ui
    port: 16686
    targetPort: 16686
  - name: collector-grpc
    port: 14250
    targetPort: 14250
  - name: collector-http
    port: 14268
    targetPort: 14268
  - name: agent-compact
    port: 6831
    protocol: UDP
    targetPort: 6831
  - name: agent-binary
    port: 6832
    protocol: UDP
    targetPort: 6832
  - name: zipkin
    port: 9411
    targetPort: 9411
  - name: otlp-grpc  # ✅ Nome correto
    port: 4317
    targetPort: 4317
  - name: otlp-http  # ✅ Nome correto
    port: 4318
    targetPort: 4318
  type: ClusterIP
````

## Aplicar a correção:

```bash
kubectl apply -f k8s/tools/deploy-jaeger.yaml

# Aguardar o pod reiniciar
kubectl rollout status deployment jaeger

# Verificar logs do Jaeger
kubectl logs -l app=jaeger | grep -i "otlp\|listening"

# Gerar tráfego novamente
curl http://127.0.0.1/app-a

# Verificar logs do collector
kubectl logs -l app.kubernetes.io/name=opentelemetry-collector --tail=50
```

Depois dessa correção, os traces devem aparecer no Jaeger UI com os nomes corretos dos serviços (app-a, app-b, app-c).





ajustado o #file:deploy-jaeger.yaml 
porém jaeger segue sem os traces no Jaeger UI com os nomes corretos dos serviços (app-a, app-b, app-c).






Agora vamos verificar se o **OpenTelemetry Collector está enviando traces para o Jaeger**. O problema pode estar no values.yaml:

## 1. Verificar logs do Collector

```bash
# Verificar se o collector está recebendo traces das apps
kubectl logs -l app.kubernetes.io/name=opentelemetry-collector --tail=100 | grep -i "trace\|span"

# Verificar se há erros ao exportar para o Jaeger
kubectl logs -l app.kubernetes.io/name=opentelemetry-collector --tail=100 | grep -i "jaeger\|error\|fail"
```

## 2. Testar gerando tráfego REAL (não localhost)

O problema é que você está testando com `curl http://localhost:8000` DENTRO do pod. Isso **não gera traces distribuídos**!

```bash
# Gerar tráfego EXTERNO que passa por app-a -> app-b -> app-c
curl http://127.0.0.1/app-a
curl http://127.0.0.1/app-a
curl http://127.0.0.1/app-a

# Aguardar alguns segundos
sleep 5

# Verificar logs do collector
kubectl logs -l app.kubernetes.io/name=opentelemetry-collector --tail=50
```

## 3. Verificar se o Jaeger está recebendo traces

```bash
# Ver logs do Jaeger
kubectl logs -l app=jaeger | grep -i "otlp\|span\|trace"

# Acessar a API do Jaeger para ver serviços
kubectl exec -it $(kubectl get pod -l app=jaeger -o name) -- wget -qO- http://localhost:16686/api/services
```

## 4. Problema provável: Resource Attributes

O values.yaml está sobrescrevendo `service.name` para `otel-collector`:

````yaml
# ...existing code...
extraEnvs:
  # ❌ REMOVER ISTO - está sobrescrevendo o service.name das apps
  # - name: OTEL_RESOURCE_ATTRIBUTES
  #   value: "service.name=otel-collector,deployment.environment=production"
# ...existing code...
````

## 5. Atualizar Helm e reiniciar pods das apps

```bash
# Atualizar o collector sem o extraEnvs
helm upgrade otel-collector open-telemetry/opentelemetry-collector --values k8s/values.yaml

# Reiniciar os pods das apps para pegar nova configuração
kubectl rollout restart deployment app-a app-b app-c

# Aguardar
kubectl rollout status deployment app-a app-b app-c

# Gerar tráfego
curl http://127.0.0.1/app-a

# Verificar Jaeger UI
minikube service jaeger --url
```

Remova o `extraEnvs` do values.yaml e teste novamente. Esse é provavelmente o problema!
















>
> kubectl logs -l app.kubernetes.io/name=opentelemetry-collector --tail=100 | grep -i "trace\|span"
> kubectl logs -l app.kubernetes.io/name=opentelemetry-collector --tail=100 | grep -i "jaeger\|error\|fail"
> kubectl logs -l app.kubernetes.io/name=opentelemetry-collector --tail=50
Timestamp: 2026-01-17 20:12:19.235791385 +0000 UTC
Value: 0.439046
Metric #2
Descriptor:
     -> Name: app_requests_total
     -> Description: Número de requisições processadas
     -> Unit: 1
     -> DataType: Sum
     -> IsMonotonic: true
     -> AggregationTemporality: Cumulative
NumberDataPoints #0
Data point attributes:
     -> app: Str(app-b)
     -> endpoint: Str(/)
StartTimestamp: 2026-01-17 20:00:47.485138834 +0000 UTC
Timestamp: 2026-01-17 20:12:19.235791385 +0000 UTC
Value: 2
Metric #3
Descriptor:
     -> Name: app_response_time_seconds
     -> Description: Tempo de resposta das requisições em segundos
     -> Unit: s
     -> DataType: Histogram
     -> AggregationTemporality: Cumulative
HistogramDataPoints #0
Data point attributes:
     -> app: Str(app-b)
     -> endpoint: Str(/)
StartTimestamp: 2026-01-17 20:00:47.485227504 +0000 UTC
Timestamp: 2026-01-17 20:12:19.235791385 +0000 UTC
Count: 2
Sum: 0.000000
Min: 0.000000
Max: 0.000000
ExplicitBounds #0: 0.005000
ExplicitBounds #1: 0.010000
ExplicitBounds #2: 0.025000
ExplicitBounds #3: 0.050000
ExplicitBounds #4: 0.100000
ExplicitBounds #5: 0.250000
ExplicitBounds #6: 0.500000
Buckets #0, Count: 2
Buckets #1, Count: 0
Buckets #2, Count: 0
Buckets #3, Count: 0
Buckets #4, Count: 0
Buckets #5, Count: 0
Buckets #6, Count: 0
Buckets #7, Count: 0
        {"resource": {"service.instance.id": "10d46f84-3d09-496a-8677-fdfb9d901426", "service.name": "otelcol-contrib", "service.version": "0.128.0"}, "otelcol.component.id": "debug", "otelcol.component.kind": "exporter", "otelcol.signal": "metrics"}
>
> kubectl logs -l app=jaeger | grep -i "otlp\|span\|trace"

 ~/cursos/opentelemetry/opentelemetry-curso-devopspro/app-telemetria  main !3   






Encontrei o problema! O Collector **NÃO está recebendo traces**, apenas métricas e logs. Veja nos logs:

```
"otelcol.signal": "metrics"
```

Não há **nenhuma menção a traces/spans** nos logs do collector!

## Problema: As aplicações não estão enviando traces

Verifique se a aplicação está configurada corretamente:

```bash
# Ver variáveis de ambiente do app-a
kubectl exec -it $(kubectl get pod -l app=app-a -o name) -- env | grep -i otel

# Ver logs do app-a para erros
kubectl logs -l app=app-a --tail=50
```

## Solução: Verificar código da aplicação

A aplicação precisa estar instrumentada para enviar traces. Verifique se o código tem:

```python
# O código precisa ter algo assim para enviar traces
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter

# Configurar o tracer
trace.set_tracer_provider(TracerProvider())
tracer = trace.get_tracer(__name__)

# Exportar para o OTLP endpoint
otlp_exporter = OTLPSpanExporter(
    endpoint=os.getenv("OTLP_ENDPOINT", "http://localhost:4318/v1/traces")
)
```

## Teste: Verificar se o endpoint está correto

```bash
# Entrar no pod do app-a
kubectl exec -it $(kubectl get pod -l app=app-a -o name) -- sh

# Verificar se consegue alcançar o collector
apt-get update && apt-get install -y curl
curl -v http://otel-collector-opentelemetry-collector:4318/v1/traces
```

## Verificar se a aplicação está configurada para enviar traces

Provavelmente a aplicação está enviando apenas **logs e métricas**, mas não **traces**. A imagem `fabriciosveronez/app-telemetria:v1` precisa estar configurada para isso.

Você pode compartilhar o código fonte da aplicação ou verificar a documentação dela?





OTLP_ENDPOINT
http://otel-collector-opentelemetry-collector:4318











root@app-a-74f7747c78-jjv2l:/app# 
root@app-a-74f7747c78-jjv2l:/app# env | grep -i otel
OTLP_ENDPOINT=http://otel-collector-opentelemetry-collector:4318
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_4318_TCP_PROTO=tcp
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_SERVICE_PORT=8889
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_4318_TCP_PORT=4318
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_4317_TCP=tcp://10.101.27.82:4317
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_4317_TCP_PROTO=tcp
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_SERVICE_HOST=10.101.27.82
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_SERVICE_PORT_OTLP=4317
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_8889_TCP_PROTO=tcp
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_8889_TCP=tcp://10.101.27.82:8889
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_8889_TCP_ADDR=10.101.27.82
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_4318_TCP_ADDR=10.101.27.82
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_4317_TCP_ADDR=10.101.27.82
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_4318_TCP=tcp://10.101.27.82:4318
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_SERVICE_PORT_METRICS=8889
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_SERVICE_PORT_OTLP_HTTP=4318
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_8889_TCP_PORT=8889
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT=tcp://10.101.27.82:8889
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_4317_TCP_PORT=4317
root@app-a-74f7747c78-jjv2l:/app# apt-get update && apt-get install -y curl
Hit:1 http://deb.debian.org/debian bookworm InRelease
Hit:2 http://deb.debian.org/debian bookworm-updates InRelease
Hit:3 http://deb.debian.org/debian-security bookworm-security InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
curl is already the newest version (7.88.1-10+deb12u14).
0 upgraded, 0 newly installed, 0 to remove and 31 not upgraded.
root@app-a-74f7747c78-jjv2l:/app# curl -v http://otel-collector-opentelemetry-collector:4318/v1/traces
*   Trying 10.101.27.82:4318...
* Connected to otel-collector-opentelemetry-collector (10.101.27.82) port 4318 (#0)
> GET /v1/traces HTTP/1.1
> Host: otel-collector-opentelemetry-collector:4318
> User-Agent: curl/7.88.1
> Accept: */*
> 
< HTTP/1.1 405 Method Not Allowed
< Content-Type: text/plain
< Date: Sat, 17 Jan 2026 20:19:06 GMT
< Content-Length: 41
< 
* Connection #0 to host otel-collector-opentelemetry-collector left intact
405 method not allowed, supported: [POST]root@app-a-74f7747c78-jjv2l:/app# 
root@app-a-74f7747c78-jjv2l:/app# 




nao tenho acesso ao codigo fonte dA imagem fabriciosveronez/app-telemetria:v1
mas deve ser o mesmo código que uso no docker com compose em anexo #file:tracing.py 
curl tá ok


root@app-a-74f7747c78-jjv2l:/app# 
root@app-a-74f7747c78-jjv2l:/app# env | grep -i otel
OTLP_ENDPOINT=http://otel-collector-opentelemetry-collector:4318
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_4318_TCP_PROTO=tcp
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_SERVICE_PORT=8889
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_4318_TCP_PORT=4318
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_4317_TCP=tcp://10.101.27.82:4317
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_4317_TCP_PROTO=tcp
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_SERVICE_HOST=10.101.27.82
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_SERVICE_PORT_OTLP=4317
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_8889_TCP_PROTO=tcp
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_8889_TCP=tcp://10.101.27.82:8889
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_8889_TCP_ADDR=10.101.27.82
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_4318_TCP_ADDR=10.101.27.82
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_4317_TCP_ADDR=10.101.27.82
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_4318_TCP=tcp://10.101.27.82:4318
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_SERVICE_PORT_METRICS=8889
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_SERVICE_PORT_OTLP_HTTP=4318
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_8889_TCP_PORT=8889
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT=tcp://10.101.27.82:8889
OTEL_COLLECTOR_OPENTELEMETRY_COLLECTOR_PORT_4317_TCP_PORT=4317
root@app-a-74f7747c78-jjv2l:/app# apt-get update && apt-get install -y curl
Hit:1 http://deb.debian.org/debian bookworm InRelease
Hit:2 http://deb.debian.org/debian bookworm-updates InRelease
Hit:3 http://deb.debian.org/debian-security bookworm-security InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
curl is already the newest version (7.88.1-10+deb12u14).
0 upgraded, 0 newly installed, 0 to remove and 31 not upgraded.
root@app-a-74f7747c78-jjv2l:/app# curl -v http://otel-collector-opentelemetry-collector:4318/v1/traces
*   Trying 10.101.27.82:4318...
* Connected to otel-collector-opentelemetry-collector (10.101.27.82) port 4318 (#0)
> GET /v1/traces HTTP/1.1
> Host: otel-collector-opentelemetry-collector:4318
> User-Agent: curl/7.88.1
> Accept: */*
> 
< HTTP/1.1 405 Method Not Allowed
< Content-Type: text/plain
< Date: Sat, 17 Jan 2026 20:19:06 GMT
< Content-Length: 41
< 
* Connection #0 to host otel-collector-opentelemetry-collector left intact
405 method not allowed, supported: [POST]root@app-a-74f7747c78-jjv2l:/app# 
root@app-a-74f7747c78-jjv2l:/app# 




Perfeito! O endpoint está OK (405 é esperado para GET, ele aceita apenas POST). O problema é que **a aplicação não está enviando traces**!

Baseado no código tracing.py, a aplicação **precisa ser chamada pelo endpoint correto** para gerar traces. Vamos testar:

## 1. Verificar qual endpoint da aplicação gera traces

```bash
# Ver logs da aplicação quando você faz requisição
kubectl logs -l app=app-a --tail=20 -f
```

Em outro terminal:

```bash
# Fazer requisição que deve gerar traces
curl http://127.0.0.1/app-a

# Se tiver um endpoint específico para testar
curl http://127.0.0.1/teste
curl http://127.0.0.1/health
```

## 2. O problema provável: Endpoint incorreto no OTLP_ENDPOINT

No código tracing.py, veja esta linha:

```python
TRACE_OTLP_ENDPOINT = f"{OTLP_ENDPOINT}/v1/traces"
```

Mas no deploy.yaml está assim:

```yaml
OTLP_ENDPOINT: "http://otel-collector-opentelemetry-collector:4318"
```

Isso resulta em: `http://otel-collector-opentelemetry-collector:4318/v1/traces` ✅

Isso está correto! Então o problema é outro...

## 3. Teste: Enviar um trace manualmente para confirmar pipeline

```bash
# Dentro do pod app-a
kubectl exec -it $(kubectl get pod -l app=app-a -o name) -- sh

# Enviar um trace de teste
curl -X POST http://otel-collector-opentelemetry-collector:4318/v1/traces \
  -H "Content-Type: application/json" \
  -d '{
    "resourceSpans": [{
      "resource": {
        "attributes": [{
          "key": "service.name",
          "value": {"stringValue": "test-service"}
        }]
      },
      "scopeSpans": [{
        "spans": [{
          "name": "test-span",
          "spanId": "0123456789abcdef",
          "traceId": "0123456789abcdef0123456789abcdef",
          "kind": 1,
          "startTimeUnixNano": "1234567890000000000",
          "endTimeUnixNano": "1234567891000000000"
        }]
      }]
    }]
  }'
```

## 4. Verificar se aparece no Jaeger

Depois de enviar o trace de teste, vá no Jaeger UI e procure pelo serviço `test-service`.

## 5. Se o trace de teste aparecer, o problema é na aplicação

A aplicação pode não estar sendo instrumentada corretamente. Verifique:

```bash
# Ver todos os logs da aplicação desde o início
kubectl logs -l app=app-a --tail=100 | grep -i "trace\|otel\|span"
```

**Qual é o endpoint que você está chamando? Você precisa chamar um endpoint que execute código instrumentado, não apenas retornar JSON simples.**











root@app-a-74f7747c78-jjv2l:/app# 
root@app-a-74f7747c78-jjv2l:/app# curl -X POST http://otel-collector-opentelemetry-collector:4318/v1/traces \
  -H "Content-Type: application/json" \
  -d '{
    "resourceSpans": [{
      "resource": {
        "attributes": [{
          "key": "service.name",
          "value": {"stringValue": "test-service"}
        }]
      },
      "scopeSpans": [{
        "spans": [{
          "name": "test-span",
          "spanId": "0123456789abcdef",
          "traceId": "0123456789abcdef0123456789abcdef",
          "kind": 1,
          "startTimeUnixNano": "1234567890000000000",
          "endTimeUnixNano": "1234567891000000000"
        }]
      }]
    }]
  }'
{"partialSuccess":{}}root@app-a-74f7747c78-jjv2l:/app# 
root@app-a-74f7747c78-jjv2l:/app# 
root@app-a-74f7747c78-jjv2l:/app# 
root@app-a-74f7747c78-jjv2l:/app# 
root@app-a-74f7747c78-jjv2l:/app# date
Sat Jan 17 20:41:34 UTC 2026
root@app-a-74f7747c78-jjv2l:/app# 






Excelente! O trace de teste foi enviado com sucesso (`{"partialSuccess":{}}`). Agora vamos verificar se ele chegou no Jaeger:

## 1. Verificar logs do Collector (deve mostrar o trace)

```bash
kubectl logs -l app.kubernetes.io/name=opentelemetry-collector --tail=50 | grep -i "test-service\|test-span"
```

## 2. Acessar Jaeger UI e procurar o serviço

```bash
# Pegar URL do Jaeger
minikube service jaeger --url

# Ou fazer port-forward
kubectl port-forward svc/jaeger 16686:16686
```

Acesse `http://localhost:16686` e procure pelo serviço **test-service**.

## 3. Se o test-service aparecer no Jaeger, o problema É A APLICAÇÃO!

A aplicação não está sendo instrumentada corretamente. Veja os logs dela:

```bash
# Ver logs completos do app-a
kubectl logs -l app=app-a --tail=100

# Procurar por erros de inicialização do OpenTelemetry
kubectl logs -l app=app-a | grep -i "error\|fail\|exception"
```

## 4. Problema provável: A aplicação não está fazendo requisições HTTP instrumentadas

Você está testando com `curl http://localhost:8000` que retorna apenas JSON. Isso **não gera traces distribuídos**!

Você precisa chamar de **FORA** do cluster para a cadeia funcionar:

```bash
# Fora do pod, do seu terminal local:
curl http://127.0.0.1/app-a

# Isso deve fazer: Cliente -> app-a -> app-b -> app-c
```

## 5. Verificar se a aplicação está realmente instrumentada

```bash
# Ver o conteúdo do container para confirmar que tem OpenTelemetry
kubectl exec -it $(kubectl get pod -l app=app-a -o name) -- ls -la /app/

# Ver se tem arquivos Python de telemetria
kubectl exec -it $(kubectl get pod -l app=app-a -o name) -- find /app -name "*otel*" -o -name "*telemetry*"
```

**Próximo passo crucial**: Chame `curl http://127.0.0.1/app-a` do seu terminal local e veja se aparecem traces no Jaeger. Se não aparecer, a aplicação não está instrumentada corretamente!



**Não**, o `minikube tunnel` não é o problema! O problema é que **a aplicação não está instrumentada ou não está sendo inicializada corretamente**.

O trace de teste funcionou, provando que o pipeline **Collector → Jaeger está OK**. O problema é na aplicação.

## Verificações críticas:

### 1. Ver logs da aplicação ao iniciar (procurar erros do OpenTelemetry)

```bash
kubectl logs -l app=app-a --tail=100 | grep -i "error\|fail\|exception"
```

### 2. Verificar se a aplicação tem os arquivos de instrumentação

```bash
kubectl exec -it $(kubectl get pod -l app=app-a -o name) -- ls -la /app/
kubectl exec -it $(kubectl get pod -l app=app-a -o name) -- ls -la /app/otel/
```

### 3. Testar fazendo requisição EXTERNA (do seu terminal local)

```bash
# Acessar app-a que deve chamar app-b que chama app-c
curl http://127.0.0.1:39253

# Ver logs IMEDIATAMENTE após
kubectl logs -l app=app-a --tail=20
kubectl logs -l app=app-b --tail=20
kubectl logs -l app=app-c --tail=20
```

### 4. Verificar se o código da aplicação está sendo executado

A aplicação pode estar retornando apenas um JSON estático sem executar o código instrumentado. Olhe os logs:

```bash
# Ver todos os logs desde o início
kubectl logs -l app=app-a --since=1h

# Ver se há menção a "tracer", "span", "OpenTelemetry"
kubectl logs -l app=app-a | grep -i "tracer\|span\|opentelemetry\|otlp"
```

### 5. Problema provável: A imagem está diferente do código local

A imagem `fabriciosveronez/app-telemetria:v1` pode **NÃO ter** o código de instrumentação que você tem em `tracing.py`.

**Solução**: Construir uma nova imagem com o código correto:

```bash
# Ir para o diretório da aplicação
cd /home/fernando/cursos/opentelemetry/opentelemetry-curso-devopspro/app-telemetria

# Build da imagem
docker build -t minha-app-telemetria:v1 .

# Carregar no Minikube
minikube image load minha-app-telemetria:v1

# Atualizar o deploy
kubectl set image deployment/app-a app=minha-app-telemetria:v1
kubectl set image deployment/app-b app=minha-app-telemetria:v1
kubectl set image deployment/app-c app=minha-app-telemetria:v1
```

**Qual é a saída de `kubectl logs -l app=app-a`?** Isso vai revelar o problema!





### 5. Problema provável: A imagem está diferente do código local

A imagem `fabriciosveronez/app-telemetria:v1` pode **NÃO ter** o código de instrumentação que você tem em `tracing.py`.

**Solução**: Construir uma nova imagem com o código correto