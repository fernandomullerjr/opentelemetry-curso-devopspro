
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