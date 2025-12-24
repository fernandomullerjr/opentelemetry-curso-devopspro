
## Adicionando o Jaeger

- No arquivo tracing.py, necessário importar:

~~~~py
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
~~~~


- Já na aula a variável OTLP_ENDPOINT aponta para a 4318 do Jaeger, que usa HTTP.
- Porém, no código atual do lab já está com o "otel-collector", onde também usa HTTP na porta 4318, mas o OTLP_ENDPOINT aponta para o otel-collector mesmo.
- Porta configurada no otel-collector:

~~~~compose
    - "4318:4318"  # OTLP HTTP receiver
~~~~