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


