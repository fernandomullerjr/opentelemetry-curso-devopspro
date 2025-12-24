## Instrumentação básica

- No arquivo 'opentelemetry-curso-devopspro/app-telemetria/src/otel/tracing.py"
importar o necessário:

~~~~py
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.resources import Resource

from opentelemetry.sdk.trace.export import (
    BatchSpanProcessor,
    ConsoleSpanExporter,
)
~~~~

### Explicação dos Imports

#### Core OpenTelemetry
```python
from opentelemetry import trace
```
- **Propósito**: Módulo principal para criação e gerenciamento de traces
- **Função**: Fornece a API para obter tracers e criar spans
- **Uso**: `trace.get_tracer(__name__)` para obter um tracer

#### TracerProvider - Fábrica de Tracers
```python
from opentelemetry.sdk.trace import TracerProvider
```
- **Propósito**: Implementação concreta que gerencia a criação de tracers
- **Responsabilidades**:
  - Configurar processadores de spans
  - Definir políticas de sampling
  - Gerenciar recursos e metadados
- **Analogia**: Como uma fábrica que produz tracers configurados

#### Resource - Metadados do Serviço
```python
from opentelemetry.sdk.resources import Resource
```
- **Propósito**: Define metadados sobre o serviço/aplicação
- **Informações incluídas**:
  - Nome do serviço (`service.name`)
  - Versão (`service.version`)
  - Ambiente de execução
  - Identificadores únicos
- **Benefício**: Facilita identificação e filtros no backend de observabilidade

#### BatchSpanProcessor - Processamento Eficiente
```python
from opentelemetry.sdk.trace.export import BatchSpanProcessor
```
- **Propósito**: Processa spans em lotes para otimizar performance
- **Vantagens**:
  - Reduz overhead de rede
  - Melhora throughput
  - Controla uso de memória
- **Configurações**: Tamanho do lote, timeout, frequência de envio

#### ConsoleSpanExporter - Debug Local
```python
from opentelemetry.sdk.trace.export import ConsoleSpanExporter
```
- **Propósito**: Exporta spans para o console (stdout)
- **Casos de uso**:
  - Desenvolvimento local
  - Debugging de instrumentação
  - Testes de validação
- **Formato**: JSON estruturado legível

### Fluxo de Configuração

```
Resource → TracerProvider → BatchSpanProcessor → ConsoleSpanExporter
    ↓           ↓                    ↓                    ↓
Metadados   Fábrica de      Processamento        Saída para
do serviço   tracers         em lotes            console
```

Esses imports formam a base para configurar um sistema de tracing completo, desde a definição dos metadados do serviço até a exportação dos dados coletados.




- Continua em:
06:50min

- No OpenTelemtry temos convenções, entre elas a de Semantica para Resources:
<https://opentelemetry.io/docs/specs/semconv/>
<https://opentelemetry.io/docs/specs/semconv/resource/>

- É necessário importar do semconv os atributos e usar de forma fácil, que ele já aplica a nomenclatura adequada desta forma:

~~~~py
from opentelemetry.semconv.attributes.service_attributes import (
    SERVICE_NAME,
    SERVICE_VERSION
)

# =============================================================================
# RECURSO (RESOURCE) - METADADOS DO SERVIÇO
# =============================================================================
# Resource: Define informações sobre o serviço que está gerando os traces
# Essas informações ajudam a identificar de qual serviço/versão vêm os dados
resource = Resource.create({
    SERVICE_NAME: APP_NAME,        # Nome do serviço
    SERVICE_VERSION: "1.0.0"       # Versão do serviço
})
~~~~

- No caso aqui seria "service.name", mas usando o SERVICE_NAME ele já aplica ajustado.

~~~~py
# =============================================================================
# TRACER PROVIDER - CONFIGURAÇÃO CENTRAL DO TRACING
# =============================================================================
# TracerProvider: É o ponto central de configuração para tracing
# Define como os traces serão processados e exportados
provider = TracerProvider(resource=resource)

# =============================================================================
# PROCESSADORES DE SPANS (SPAN PROCESSORS)
# =============================================================================
# Span Processor: Define como os spans (segmentos de trace) serão processados
# BatchSpanProcessor: Agrupa spans em lotes para envio eficiente

# Processador para console - útil para desenvolvimento/debug
# Envia os traces para o console (terminal) para visualização local
processor_console = BatchSpanProcessor(ConsoleSpanExporter())
~~~~


- Configurando o processador, para onde iremos enviar os Spans.
- É possível setar mais de um Processor, ex: enviar para a Console e para o Sistema de Observabilidade.
- Na configuração abaixo foi setado para enviar para o Sistema de Observabilidade:

~~~~py
# =============================================================================
# CONFIGURAÇÃO DOS PROCESSADORES
# =============================================================================
# Comentado o processador de console para não poluir o terminal
# provider.add_span_processor(processor_console)

# Adiciona o processador OTLP que enviará os traces para o sistema de observabilidade
provider.add_span_processor(processor_otlp)
~~~~


- Configuração do Tracer, que é necessário com o nome:

~~~~py
# =============================================================================
# TRACER - INSTRUMENTO PRINCIPAL PARA CRIAR SPANS
# =============================================================================
# Tracer: É o instrumento principal para criar spans (segmentos de trace)
# Cada serviço deve ter seu próprio tracer identificado pelo nome
tracer = trace.get_tracer(APP_NAME)
~~~~




## PENDENTE
- COntinua em:
16:01min
na parte de configuração do primeiro span
"with tracer.start_as_current_span("process-request", context=context) as main_span:"



## Dia 24/12/2025

- COntinua em:
16:01min


Lá no arquivo "opentelemetry-curso-devopspro/app-telemetria/src/app.py"
tem o trecho onde é configurado o Span principal:

~~~~py

    with tracer.start_as_current_span("process-request", context=context) as main_span:

### Restante do código ...
~~~~



Lá no arquivo "opentelemetry-curso-devopspro/app-telemetria/src/app.py"
tem o trecho onde é configurado o Span filho também:

~~~~py

                with tracer.start_as_current_span("send-request") as child_span:

                    try:
                        # Log do início da requisição externa
                        logger.debug(
                            "Enviando requisição para serviço downstream",
                            extra={
                                **log_context,
                                "destination_url": url,
                                "request_method": "POST",
                                "request_path": "/process"
                            }
                        )

                        headers = {}
                        propagator.inject(headers)

                        child_span.set_attribute("net.peer.name", url)
                        child_span.set_attribute("destination.url", url)

                        child_span.add_event("Requisição externa bem-sucedida", {"url": url})

                        resp = requests.post(
                            f"{url}/process",
                            json=original_payload,
                            headers=headers,
                            timeout=5
                        )

                        child_span.set_attribute("http.status_code", resp.status_code)

~~~~





- Gerada request via:
opentelemetry-curso-devopspro/app-telemetria/teste_request.http

~~~~sh
HTTP/1.1 200 OK
date: Wed, 24 Dec 2025 20:06:49 GMT
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
~~~~


- Verificando os logs do Container do app-a:
temos detalhes do trace e span

~~~~bash

> docker compose logs app-a
app-a-1  | INFO:     Started server process [1]
app-a-1  | INFO:     Waiting for application startup.
app-a-1  | INFO:     Application startup complete.
app-a-1  | INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
app-a-1  | INFO:     172.19.0.1:46454 - "GET /process HTTP/1.1" 405 Method Not Allowed
app-a-1  | INFO:     172.19.0.1:46456 - "GET /favicon.ico HTTP/1.1" 404 Not Found
app-a-1  | INFO:     172.19.0.1:46454 - "GET /metrics HTTP/1.1" 200 OK
app-a-1  | {
app-a-1  |     "body": "Endpoint raiz acessado",
app-a-1  |     "severity_number": 9,
app-a-1  |     "severity_text": "INFO",
app-a-1  |     "attributes": {
app-a-1  |         "service_name": "app-a",
app-a-1  |         "endpoint": "/",
app-a-1  |         "operation": "health_check",
app-a-1  |         "code.file.path": "/app/app.py",
app-a-1  |         "code.function.name": "read_root",
app-a-1  |         "code.line.number": 26
app-a-1  |     },
app-a-1  |     "dropped_attributes": 0,
app-a-1  |     "timestamp": "2025-12-24T19:43:08.343987Z",
app-a-1  |     "observed_timestamp": "2025-12-24T19:43:08.344109Z",
app-a-1  |     "trace_id": "0x00000000000000000000000000000000",
app-a-1  |     "span_id": "0x0000000000000000",
app-a-1  |     "trace_flags": 0,
app-a-1  |     "resource": {
app-a-1  |         "attributes": {
app-a-1  |             "telemetry.sdk.language": "python",
app-a-1  |             "telemetry.sdk.name": "opentelemetry",
app-a-1  |             "telemetry.sdk.version": "1.34.1",
app-a-1  |             "service.name": "app-a",
app-a-1  |             "service.version": "1.0.0"
app-a-1  |         },
app-a-1  |         "schema_url": ""
app-a-1  |     }
app-a-1  | }
app-a-1  | INFO:     172.19.0.1:46454 - "GET / HTTP/1.1" 200 OK
app-a-1  | INFO:     172.19.0.1:46456 - "GET /favicon.ico HTTP/1.1" 404 Not Found
app-a-1  | {
app-a-1  |     "body": "Iniciando processamento de requisi\u00e7\u00e3o",
app-a-1  |     "severity_number": 9,
app-a-1  |     "severity_text": "INFO",
app-a-1  |     "attributes": {
app-a-1  |         "service_name": "app-a",
app-a-1  |         "operation": "process_request",
app-a-1  |         "payload": [],
app-a-1  |         "payload_size": 0,
app-a-1  |         "app_config": {
app-a-1  |             "error_rate": 5,
app-a-1  |             "max_latency": 100,
app-a-1  |             "destinations": [
app-a-1  |                 "http://app-b:8000",
app-a-1  |                 "http://app-c:8000"
app-a-1  |             ]
app-a-1  |         },
app-a-1  |         "code.file.path": "/app/app.py",
app-a-1  |         "code.function.name": "process_request",
app-a-1  |         "code.line.number": 80
app-a-1  |     },
app-a-1  |     "dropped_attributes": 0,
app-a-1  |     "timestamp": "2025-12-24T20:06:49.508117Z",
app-a-1  |     "observed_timestamp": "2025-12-24T20:06:49.508226Z",
app-a-1  |     "trace_id": "0x00000000000000000000000000000000",
app-a-1  |     "span_id": "0x0000000000000000",
app-a-1  |     "trace_flags": 0,
app-a-1  |     "resource": {
app-a-1  |         "attributes": {
app-a-1  |             "telemetry.sdk.language": "python",
app-a-1  |             "telemetry.sdk.name": "opentelemetry",
app-a-1  |             "telemetry.sdk.version": "1.34.1",
app-a-1  |             "service.name": "app-a",
app-a-1  |             "service.version": "1.0.0"
app-a-1  |         },
app-a-1  |         "schema_url": ""
app-a-1  |     }
app-a-1  | }
app-a-1  | 2025-12-24 20:06:49,510 - opentelemetry.attributes - 1 - WARNING - Invalid type HttpRequestMethodValues for attribute 'http.request.method' value. Expected one of ['bool', 'str', 'bytes', 'int', 'float'] or a sequence of those types
app-a-1  | {
app-a-1  |     "body": "Iniciando propaga\u00e7\u00e3o para servi\u00e7os downstream",
app-a-1  |     "severity_number": 9,
app-a-1  |     "severity_text": "INFO",
app-a-1  |     "attributes": {
app-a-1  |         "service_name": "app-a",
app-a-1  |         "operation": "process_request",
app-a-1  |         "destination_urls": [
app-a-1  |             "http://app-b:8000",
app-a-1  |             "http://app-c:8000"
app-a-1  |         ],
app-a-1  |         "destinations_count": 2,
app-a-1  |         "payload_to_send": [
app-a-1  |             "app-a"
app-a-1  |         ],
app-a-1  |         "code.file.path": "/app/app.py",
app-a-1  |         "code.function.name": "process_request",
app-a-1  |         "code.line.number": 185
app-a-1  |     },
app-a-1  |     "dropped_attributes": 0,
app-a-1  |     "timestamp": "2025-12-24T20:06:49.565143Z",
app-a-1  |     "observed_timestamp": "2025-12-24T20:06:49.565185Z",
app-a-1  |     "trace_id": "0xa53456b679eb3442e53ca118f78b8caf",
app-a-1  |     "span_id": "0x347a825f57032c9c",
app-a-1  |     "trace_flags": 1,
app-a-1  |     "resource": {
app-a-1  |         "attributes": {
app-a-1  |             "telemetry.sdk.language": "python",
app-a-1  |             "telemetry.sdk.name": "opentelemetry",
app-a-1  |             "telemetry.sdk.version": "1.34.1",
app-a-1  |             "service.name": "app-a",
app-a-1  |             "service.version": "1.0.0"
app-a-1  |         },
app-a-1  |         "schema_url": ""
app-a-1  |     }
app-a-1  | }
app-a-1  | {
app-a-1  |     "body": "Requisi\u00e7\u00e3o externa bem-sucedida",
app-a-1  |     "severity_number": 9,
app-a-1  |     "severity_text": "INFO",
app-a-1  |     "attributes": {
app-a-1  |         "service_name": "app-a",
app-a-1  |         "operation": "process_request",
app-a-1  |         "destination_url": "http://app-b:8000",
app-a-1  |         "response_status": 200,
app-a-1  |         "payload_sent": [
app-a-1  |             "app-a",
app-a-1  |             "app-b",
app-a-1  |             "app-c"
app-a-1  |         ],
app-a-1  |         "code.file.path": "/app/app.py",
app-a-1  |         "code.function.name": "process_request",
app-a-1  |         "code.line.number": 232
app-a-1  |     },
app-a-1  |     "dropped_attributes": 0,
app-a-1  |     "timestamp": "2025-12-24T20:06:49.836700Z",
app-a-1  |     "observed_timestamp": "2025-12-24T20:06:49.836749Z",
app-a-1  |     "trace_id": "0xa53456b679eb3442e53ca118f78b8caf",
app-a-1  |     "span_id": "0xf72e5c478b7d1fc2",
app-a-1  |     "trace_flags": 1,
app-a-1  |     "resource": {
app-a-1  |         "attributes": {
app-a-1  |             "telemetry.sdk.language": "python",
app-a-1  |             "telemetry.sdk.name": "opentelemetry",
app-a-1  |             "telemetry.sdk.version": "1.34.1",
app-a-1  |             "service.name": "app-a",
app-a-1  |             "service.version": "1.0.0"
app-a-1  |         },
app-a-1  |         "schema_url": ""
app-a-1  |     }
app-a-1  | }
app-a-1  | {
app-a-1  |     "body": "Requisi\u00e7\u00e3o externa bem-sucedida",
app-a-1  |     "severity_number": 9,
app-a-1  |     "severity_text": "INFO",
app-a-1  |     "attributes": {
app-a-1  |         "service_name": "app-a",
app-a-1  |         "operation": "process_request",
app-a-1  |         "destination_url": "http://app-c:8000",
app-a-1  |         "response_status": 200,
app-a-1  |         "payload_sent": [
app-a-1  |             "app-a",
app-a-1  |             "app-b",
app-a-1  |             "app-c",
app-a-1  |             "app-c"
app-a-1  |         ],
app-a-1  |         "code.file.path": "/app/app.py",
app-a-1  |         "code.function.name": "process_request",
app-a-1  |         "code.line.number": 232
app-a-1  |     },
app-a-1  |     "dropped_attributes": 0,
app-a-1  |     "timestamp": "2025-12-24T20:06:49.894578Z",
app-a-1  |     "observed_timestamp": "2025-12-24T20:06:49.894622Z",
app-a-1  |     "trace_id": "0xa53456b679eb3442e53ca118f78b8caf",
app-a-1  |     "span_id": "0x4268474eed995d71",
app-a-1  |     "trace_flags": 1,
app-a-1  |     "resource": {
app-a-1  |         "attributes": {
app-a-1  |             "telemetry.sdk.language": "python",
app-a-1  |             "telemetry.sdk.name": "opentelemetry",
app-a-1  |             "telemetry.sdk.version": "1.34.1",
app-a-1  |             "service.name": "app-a",
app-a-1  |             "service.version": "1.0.0"
app-a-1  |         },
app-a-1  |         "schema_url": ""
app-a-1  |     }
app-a-1  | }
app-a-1  | {
app-a-1  |     "body": "Processamento concluido com sucesso",
app-a-1  |     "severity_number": 9,
app-a-1  |     "severity_text": "INFO",
app-a-1  |     "attributes": {
app-a-1  |         "service_name": "app-a",
app-a-1  |         "operation": "process_request",
app-a-1  |         "result_payload": [
app-a-1  |             "app-a",
app-a-1  |             "app-b",
app-a-1  |             "app-c",
app-a-1  |             "app-c"
app-a-1  |         ],
app-a-1  |         "processing_duration": 0.38544535636901855,
app-a-1  |         "latency_simulation": 100,
app-a-1  |         "destinations_count": 2,
app-a-1  |         "code.file.path": "/app/app.py",
app-a-1  |         "code.function.name": "process_request",
app-a-1  |         "code.line.number": 281
app-a-1  |     },
app-a-1  |     "dropped_attributes": 0,
app-a-1  |     "timestamp": "2025-12-24T20:06:49.896056Z",
app-a-1  |     "observed_timestamp": "2025-12-24T20:06:49.896079Z",
app-a-1  |     "trace_id": "0xa53456b679eb3442e53ca118f78b8caf",
app-a-1  |     "span_id": "0x347a825f57032c9c",
app-a-1  |     "trace_flags": 1,
app-a-1  |     "resource": {
app-a-1  |         "attributes": {
app-a-1  |             "telemetry.sdk.language": "python",
app-a-1  |             "telemetry.sdk.name": "opentelemetry",
app-a-1  |             "telemetry.sdk.version": "1.34.1",
app-a-1  |             "service.name": "app-a",
app-a-1  |             "service.version": "1.0.0"
app-a-1  |         },
app-a-1  |         "schema_url": ""
app-a-1  |     }
app-a-1  | }
app-a-1  | INFO:     172.19.0.1:53994 - "POST /process HTTP/1.1" 200 OK

 ~/cursos/op/opentelemetry-curso-devopspro/app-telemetria  main !1   
~~~~



- É possível verificar que o Trace traz informações como:
Attributes
service.name
service.version
opentelemetry