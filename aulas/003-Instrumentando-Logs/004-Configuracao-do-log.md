
## Configuração do Log

- Arquivo onde iremos configurar:
opentelemetry-curso-devopspro/app-telemetria/src/otel/logs.py

~~~~py
# Importação das bibliotecas necessárias
import logging
import config
from opentelemetry.sdk._logs import LoggerProvider, LoggingHandler
from opentelemetry.sdk._logs.export import ConsoleLogExporter, SimpleLogRecordProcessor
from opentelemetry.sdk.resources import Resource
from opentelemetry.semconv.attributes.service_attributes import (
    SERVICE_NAME,
    SERVICE_VERSION
)
from opentelemetry._logs import set_logger_provider
from opentelemetry.exporter.otlp.proto.http._log_exporter import OTLPLogExporter
~~~~


- Observação:
o underscore diz que aquela biblioteca está em construção / desenvolvimento

## CONTINUA EM:
08:11min


## Dia 31/12/2025

- Sobre o Resource

~~~~py
# Cria um recurso OpenTelemetry com informações do serviço
# Isso ajuda a identificar a origem dos logs
resource = Resource.create({
    SERVICE_NAME: config.APP_NAME,  # Nome do serviço
    SERVICE_VERSION: "1.0.0"  # Versão do serviço
})
~~~~


- Aqui foi hard-coded, mas pode ser obtido de uma variável, etc
importante é ter a informação!




- Sobre o LoggerProvider:

~~~~py
# Inicializa o provedor de logs do OpenTelemetry
# Este é o componente principal que gerencia os logs
logger_provider = LoggerProvider(resource=resource)
~~~~

Ele cria o “provedor” de logs do OpenTelemetry para sua aplicação:

- resource define metadados do serviço (service.name e service.version) que serão anexados a cada log.
- LoggerProvider(resource=resource) inicializa o pipeline de logs com esses metadados.
- Em seguida (nas linhas abaixo), você adiciona processadores que enviam os logs para o console e para o OTLP, e registra esse provider como global com set_logger_provider.
- O LoggingHandler usa esse provider para transformar logs do logging padrão (logger.info, etc.) em LogRecords OpenTelemetry com o resource, que são então exportados pelos processadores configurados.


- Sobre o ConsoleLogExporter:

~~~~py
# Configura o exportador de logs para console
# Isso permite que os logs sejam exibidos no terminal
console_exporter = ConsoleLogExporter()
logger_provider.add_log_record_processor(
    SimpleLogRecordProcessor(console_exporter)
)

# Define o provedor de logs como global
# Isso permite que outros componentes da aplicação usem o mesmo provedor
set_logger_provider(logger_provider)

# Cria um handler OpenTelemetry para processar os logs
# Este handler integra o logging padrão do Python com OpenTelemetry
otel_handler = LoggingHandler(logger_provider=logger_provider)
otel_handler.setLevel(logging.INFO)
~~~~

Ele conecta o provedor de logs ao console:

- ConsoleLogExporter: formata e escreve cada LogRecord no stdout (terminal), útil para debug.
- add_log_record_processor(SimpleLogRecordProcessor(console_exporter)): registra um processador que envia cada log imediatamente ao exporter, de forma síncrona (sem buffer ou retentativas).

Resultado: sempre que seu logger emitir um log, ele aparecerá no terminal.



- Trecho para configurar o Exporter para OTLP:

~~~~py
# Configura o exportador de logs para OTLP
otlp_exporter = OTLPLogExporter(
    endpoint=f"{config.OTLP_ENDPOINT}/v1/logs" 
)
logger_provider.add_log_record_processor(
    SimpleLogRecordProcessor(otlp_exporter)
)
~~~~



- Configurando formato e nível dos logs
configuração para evitar logs duplicados

~~~~py
# Configura o logging básico do Python para console
# Define o formato e nível dos logs
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(name)s - %(process)d - %(levelname)s - %(message)s",
)

# Cria um logger específico para a aplicação
# Este logger será usado para registrar eventos da aplicação
logger = logging.getLogger(config.APP_NAME)
logger.addHandler(otel_handler)
logger.setLevel(logging.INFO)

# Desativa a propagação dos logs para o root logger
# Isso evita logs duplicados
logger.propagate = False
~~~~

Ele configura o logger da aplicação e evita duplicidade no console:

- logging.basicConfig(...): configura o root logger para imprimir no console com nível INFO e formato definido.
- logger = logging.getLogger(config.APP_NAME): obtém um logger nomeado da aplicação.
- logger.addHandler(otel_handler): envia os logs desse logger para o pipeline do OpenTelemetry (ConsoleLogExporter e OTLP).
- logger.setLevel(logging.INFO): define o nível mínimo desse logger.
- logger.propagate = False: impede que os logs “subam” para o root logger, evitando que a mesma mensagem apareça duas vezes no console.



como o OpenTelemetry tem 2 exportadores (ConsoleLogExporter e OTLP) chamando apenas 1 Handler?
O handler único alimenta múltiplos exporters através da arquitetura do LoggerProvider:

```
logger.info("mensagem")
         ↓
    otel_handler (1 handler)
         ↓
  logger_provider (distribui para N processors)
         ↓                    ↓
SimpleLogRecordProcessor  SimpleLogRecordProcessor
         ↓                    ↓
 ConsoleLogExporter      OTLPLogExporter
         ↓                    ↓
      terminal            collector
```

**Fluxo:**
1. LoggingHandler captura o log do Python e cria um LogRecord OpenTelemetry
2. LoggerProvider recebe o LogRecord e o envia para **todos** os processors registrados
3. Cada processor entrega ao seu exporter de forma independente

**Vantagens:**
- Um único ponto de entrada (handler)
- Múltiplos destinos configuráveis
- Fácil adicionar/remover exporters sem tocar no código da aplicação

**Dica:** Para produção, considere trocar `SimpleLogRecordProcessor` por `BatchLogRecordProcessor` para melhor performance.



- Este trecho registra um log estruturado quando o endpoint raiz é acessado:

```python
logger.info(
    "Endpoint raiz acessado",  # Mensagem principal
    extra={                     # Campos estruturados adicionais
        "service_name": config.APP_NAME,
        "endpoint": "/",
        "operation": "health_check"
    }
)
```

**O que acontece:**
- `logger.info()`: emite log de nível INFO
- `extra={}`: adiciona campos personalizados ao LogRecord do OpenTelemetry
- Esses campos aparecem como atributos estruturados no backend (não apenas na mensagem de texto)

**Resultado:**
- Console: mensagem formatada pelo `logging.basicConfig`
- OTLP/collector: LogRecord com `service.name`, `endpoint` e `operation` como atributos pesquisáveis/filtráveis

**Benefício:** permite queries como "mostre todos os logs da operation=health_check" no seu sistema de observabilidade.




- Para correlacionar os Logs e os Traces:

~~~~py

    # Obter informações do span atual para correlação
    current_span = get_current_span()
    span_context = current_span.get_span_context()
    
    # Criar contexto base para logs
    log_context = {
        "service_name": config.APP_NAME,
        "operation": "process_request"
    }
    
    # Adicionar informações de trace se disponíveis
    if span_context.is_valid:
        log_context.update({
            "trace_id": format(span_context.trace_id, "032x"),
            "span_id": format(span_context.span_id, "016x")
        })

    # Log de início do processamento com detalhes da configuração
    logger.info(
        "Iniciando processamento de requisição",
        extra={
            **log_context,
            "payload": payload.copy(),
            "payload_size": len(payload),
            "app_config": {
                "error_rate": config.APP_ERRORS,
                "max_latency": config.APP_LATENCY,
                "destinations": config.APP_URL_DESTINO.split(',') if config.APP_URL_DESTINO else []
            }
        }
    )
~~~~


# Explicação do Trecho de Código

Este trecho implementa **correlação de logs com traces** usando OpenTelemetry. Veja os pontos importantes:

## 🎯 Objetivo Principal

Correlacionar logs estruturados com traces distribuídos, permitindo rastrear requisições através de múltiplos serviços.

## 📋 Componentes Chave

### 1. **Extração do Span Atual**
```python
current_span = get_current_span()
span_context = current_span.get_span_context()
```
- Obtém o span ativo da requisição
- Extrai o contexto com informações de rastreamento

### 2. **Criação do Contexto Base**
```python
log_context = {
    "service_name": config.APP_NAME,
    "operation": "process_request"
}
```
- Informações básicas do serviço e operação
- Base para todos os logs da função

### 3. **Adição de IDs de Trace**
```python
if span_context.is_valid:
    log_context.update({
        "trace_id": format(span_context.trace_id, "032x"),
        "span_id": format(span_context.span_id, "016x")
    })
```
- ✅ **Valida** se o span está ativo
- 📍 **trace_id** (032x): identificador único da transação completa
- 📍 **span_id** (016x): identificador único desta operação específica

### 4. **Log Estruturado com Contexto**
```python
logger.info(
    "Iniciando processamento de requisição",
    extra={
        **log_context,  # Inclui trace_id e span_id
        "payload": payload.copy(),
        "payload_size": len(payload),
        "app_config": {...}
    }
)
```

## ⚠️ Informações Importantes

| Aspecto | Detalhe |
|---------|---------|
| **Formato dos IDs** | Hexadecimal com zeros à esquerda (032x = 32 chars, 016x = 16 chars) |
| **Validação** | Sempre verificar `span_context.is_valid` antes de usar |
| **Propagação** | Permite correlacionar logs entre múltiplos serviços |
| **Observabilidade** | Facilita debug ao ligar logs → traces → métricas |
| **Performance** | `.copy()` no payload pode impactar em payloads grandes |

## 🔍 Benefícios

- ✨ Rastreamento end-to-end de requisições
- 🔗 Correlação automática entre logs e traces
- 🐛 Debug facilitado em ambientes distribuídos
- 📊 Visualização completa no backend (Jaeger, Tempo, etc.)






### IMPORTANTE
- Lembrar sobre a importancia de criar o bloco do Resource, pois ajuda a identificar a origem dos logs.
