
## OpenTelemetry Collector no Docker

- Configurar um YAML do Collector:

~~~~yaml
receivers:
  otlp:
    protocols:
      http:
        endpoint: 0.0.0.0:4318
      grpc:
        endpoint: 0.0.0.0:4317

processors:
  batch: 
    timeout: 10s
    send_batch_size: 1024

exporters:
  debug:
  prometheus:
    endpoint: 0.0.0.0:8889
  otlp/jaeger:
    endpoint: jaeger:4317
    tls:
      insecure: true
  otlphttp:
    endpoint: http://loki:3100/otlp

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [otlp/jaeger, debug]
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [prometheus, debug]
    logs:
      receivers: [otlp]
      processors: [batch]
      exporters: [otlphttp, debug]
~~~~

Este arquivo é a configuração do OpenTelemetry Collector, definindo como ele recebe, processa e exporta traces, métricas e logs.

- Receivers (entrada OTLP):
  - HTTP em 0.0.0.0:4318
  - gRPC em 0.0.0.0:4317
  - Aceitam traces, métricas e logs enviados pelos serviços via OTLP.

- Processor:
  - batch: agrupa dados antes de enviar (timeout 10s, até 1024 itens) para eficiência.

- Exporters (saídas):
  - debug: imprime os dados no log do Collector (útil para debug).
  - prometheus:
    - endpoint: 0.0.0.0:8889
    - expõe as métricas para “pull” pelo Prometheus em /metrics.
  - otlp/jaeger:
    - endpoint: jaeger:4317 (gRPC), tls.insecure: true
    - envia traces para o Jaeger.
  - otlphttp:
    - endpoint: http://loki:3100/otlp
    - envia logs (formato OTLP/HTTP) para o Loki.

- Service/pipelines:
  - traces: recebe via OTLP → processa com batch → exporta para Jaeger e debug.
  - metrics: recebe via OTLP → processa com batch → expõe via Prometheus e debug.
  - logs: recebe via OTLP → processa com batch → exporta para Loki (otlphttp) e debug.

Observações:
- 0.0.0.0 significa que o Collector escuta em todas as interfaces (acessível por outros containers).
- O endpoint do exporter Prometheus segue a recomendação: apenas host:porta; o path padrão /metrics é implícito.


### Sobre o trecho do Prometheus nos Exporters

A documentação do Prometheus/OTel para o prometheus exporter recomenda:

- Endpoint apenas como host:porta, sem esquema e sem path (ex.: 0.0.0.0:8889).
- O path padrão é /metrics; não o inclua no endpoint. Se precisar mudar, ajuste no Prometheus (metrics_path), não no exporter.
- Use 0.0.0.0 para expor para outras máquinas/containers; use localhost para acesso local.

Seu endpoint 0.0.0.0:8889 está conforme a recomendação.

Documentação do Prometheus indica usar o endpoint do próprio Prometheus ao invés do Endpoint do OTEL.



## CONTINUA EM:
16:01min



- Pode ser interessante adicionar o Exporter de debug e configurar ele no pipeline

~~~~yaml
exporters:
  debug:
  prometheus:
    endpoint: 0.0.0.0:8889
  otlp/jaeger:
    endpoint: jaeger:4317
    tls:
      insecure: true
  otlphttp:
    endpoint: http://loki:3100/otlp

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [otlp/jaeger, debug]
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [prometheus, debug]
    logs:
      receivers: [otlp]
      processors: [batch]
      exporters: [otlphttp, debug]
~~~~



tenho grafana no docker
mas ao abrir http://localhost:3000
ta abrindo o portal do open webui, mas ele ta parado
verificar porque abre ele 


http://localhost:3000


> docker ps -q | xargs -I{} docker inspect -f '{{.Name}}  {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}  {{.Config.Image}}' {}
/app-telemetria-app-a-1  172.19.0.9  fabriciosveronez/app-telemetria:v1
/app-telemetria-app-c-1  172.19.0.8  app-telemetria-app-c
/app-telemetria-app-b-1  172.19.0.7  app-telemetria-app-b
/app-telemetria-otel-collector-1  172.19.0.6  otel/opentelemetry-collector-contrib:0.128.0
/app-telemetria-grafana-1  172.19.0.5  grafana/grafana:latest
/app-telemetria-loki-1  172.19.0.3  grafana/loki:latest
/prometheus  172.19.0.2  prom/prometheus:latest
/jaeger  172.19.0.4  jaegertracing/all-in-one:latest
> docker network ls
NETWORK ID     NAME                              DRIVER    SCOPE
e8f02668e272   app-telemetria_app-network        bridge    local
f025c0f13727   bridge                            bridge    local
ea82a532b0f5   host                              host      local
fe48cb64726b   none                              null      local
1657fac8bfc5   self-hosted-ai-starter-kit_demo   bridge    local
> docker network inspect bridge | grep -n "172.18" -n
> for n in $(docker network ls --format '{{.Name}}'); do
  echo "=== $n ==="
  docker network inspect "$n" --format '{{range $id,$c := .Containers}}{{printf "%s  %s\n" $c.Name $c.IPv4Address}}{{end}}'
done

=== app-telemetria_app-network ===
app-telemetria-app-b-1  172.19.0.7/16
app-telemetria-app-a-1  172.19.0.9/16
app-telemetria-loki-1  172.19.0.3/16
jaeger  172.19.0.4/16
prometheus  172.19.0.2/16
app-telemetria-app-c-1  172.19.0.8/16
app-telemetria-grafana-1  172.19.0.5/16
app-telemetria-otel-collector-1  172.19.0.6/16

=== bridge ===

=== host ===

=== none ===

=== self-hosted-ai-starter-kit_demo ===

> docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Image}}\t{{.Ports}}"
CONTAINER ID   NAMES                             IMAGE                                          PORTS
17fdafb5252c   app-telemetria-app-a-1            fabriciosveronez/app-telemetria:v1             0.0.0.0:8000->8000/tcp
d7a9e8f12cf0   app-telemetria-app-c-1            app-telemetria-app-c                           0.0.0.0:8002->8000/tcp
083e26c7375f   app-telemetria-app-b-1            app-telemetria-app-b                           0.0.0.0:8001->8000/tcp
ecd341c80ed7   app-telemetria-otel-collector-1   otel/opentelemetry-collector-contrib:0.128.0   0.0.0.0:4317-4318->4317-4318/tcp, 0.0.0.0:8889->8889/tcp, 55678-55679/tcp
e270f4748d92   app-telemetria-grafana-1          grafana/grafana:latest                         0.0.0.0:3000->3000/tcp
3778824a249f   app-telemetria-loki-1             grafana/loki:latest                            0.0.0.0:3100->3100/tcp
96fdcafdb849   prometheus                        prom/prometheus:latest                         0.0.0.0:9090->9090/tcp
887999dcb557   jaeger                            jaegertracing/all-in-one:latest                4317-4318/tcp, 0.0.0.0:14250->14250/tcp, 0.0.0.0:14268->14268/tcp, 9411/tcp, 0.0.0.0:6831-6832->6831-6832/udp, 0.0.0.0:16686->16686/tcp
> docker context ls
docker context show

NAME            DESCRIPTION                               DOCKER ENDPOINT                             ERROR
default *       Current DOCKER_HOST based configuration   unix:///var/run/docker.sock
desktop-linux   Docker Desktop                            npipe:////./pipe/dockerDesktopLinuxEngine
default
> ps -ef | grep -i 3000
root      1308   605  0 20:25 ?        00:00:00 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 3000 -container-ip 172.18.0.2 -container-port 8080
fernando 39372 27829  0 21:41 pts/11   00:00:00 grep -i 3000
> kill -9 1308
kill: kill 1308 failed: operation not permitted
> sudo kill -9 1308
> ps -ef | grep -i 3000
fernando 39565 27829  0 21:41 pts/11   00:00:00 grep -i 3000
>
>
>



- Resolvido problema da 3000 após kill no processo que fazia proxy da porta 3000 para ip de container obsoleto, que jogava para o portal Open WebUI.




## CONTINUA EM:
28:14min

explorar os data source
http://localhost:3000/connections/datasources