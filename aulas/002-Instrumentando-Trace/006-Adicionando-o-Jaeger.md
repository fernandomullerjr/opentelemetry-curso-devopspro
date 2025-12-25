
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


05:10min


- 

~~~~py
# Processador para OTLP - envia traces para sistema de observabilidade
# Este é o processador usado em produção
processor_otlp = BatchSpanProcessor(OTLPSpanExporter(endpoint=TRACE_OTLP_ENDPOINT))

# =============================================================================
# CONFIGURAÇÃO DOS PROCESSADORES
# =============================================================================
# Comentado o processador de console para não poluir o terminal
# provider.add_span_processor(processor_console)

# Adiciona o processador OTLP que enviará os traces para o sistema de observabilidade
provider.add_span_processor(processor_otlp)
~~~~



16686

http://localhost:16686



HTTP/1.1 502 Bad Gateway
date: Thu, 25 Dec 2025 17:29:28 GMT
server: uvicorn
content-length: 54
content-type: application/json
connection: close

{
  "error": "Erro ao enviar para http://app-c:8000: 500"
}



HTTP/1.1 200 OK
date: Thu, 25 Dec 2025 17:30:01 GMT
server: uvicorn
content-length: 33
content-type: application/json
connection: close

[
  "app-a",
  "app-b",
  "app-c",
  "app-c"
]



HTTP/1.1 500 Internal Server Error
date: Thu, 25 Dec 2025 17:30:42 GMT
server: uvicorn
content-length: 34
content-type: application/json
connection: close

{
  "error": "Erro simulado em app-a"
}