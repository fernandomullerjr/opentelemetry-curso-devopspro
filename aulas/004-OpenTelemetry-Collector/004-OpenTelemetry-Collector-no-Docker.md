
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