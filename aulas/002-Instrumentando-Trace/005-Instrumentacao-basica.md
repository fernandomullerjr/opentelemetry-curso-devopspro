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


