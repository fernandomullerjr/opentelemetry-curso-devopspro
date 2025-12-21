# Conceitos Básicos de Tracing

## O que é Distributed Tracing?

Distributed Tracing é uma técnica de observabilidade que permite rastrear requisições através de múltiplos serviços em sistemas distribuídos. Ele fornece visibilidade completa sobre como uma requisição flui através da arquitetura, permitindo identificar gargalos, latências e falhas.

## Componentes Fundamentais

### 1. Trace
- **Definição**: Representa uma requisição completa através do sistema
- **Características**: 
  - Possui um ID único (Trace ID)
  - Contém múltiplos spans relacionados
  - Mostra o caminho completo da requisição

### 2. Span
- **Definição**: Representa uma operação individual dentro de um trace
- **Características**:
  - Possui um ID único (Span ID)
  - Tem início e fim bem definidos
  - Contém metadados sobre a operação
  - Pode ter spans filhos (child spans)

### 3. Context
- **Definição**: Mecanismo que propaga informações do trace entre serviços
- **Função**: Garante que spans relacionados sejam corretamente associados ao trace pai

## Anatomia de um Span

```
Span: "database-query"
├── Span ID: 2c3f4a1b
├── Trace ID: 1a2b3c4d
├── Parent Span ID: 9z8y7x6w
├── Start Time: 2025-01-15T10:30:00.123Z
├── End Time: 2025-01-15T10:30:00.456Z
├── Duration: 333ms
├── Status: OK
└── Attributes:
    ├── db.system: "postgresql"
    ├── db.statement: "SELECT * FROM users WHERE id = $1"
    └── db.rows_affected: 1
```

## Tipos de Spans

### 1. Root Span
- Primeiro span de um trace
- Representa o ponto de entrada da requisição
- Não possui span pai

### 2. Child Span
- Span que possui um span pai
- Representa operações realizadas dentro do contexto do span pai

### 3. Internal Span
- Operações internas do serviço
- Exemplo: processamento de dados, cálculos

### 4. Client Span
- Chamadas para serviços externos
- Exemplo: chamadas HTTP, queries de banco

### 5. Server Span
- Processamento de requisições recebidas
- Exemplo: endpoints HTTP, consumers de mensagens

## Relações Entre Spans

### Relação Pai-Filho (Parent-Child)
```
Root Span: "HTTP GET /api/users"
├── Child Span: "authenticate-user"
├── Child Span: "query-database"
│   └── Child Span: "execute-sql"
└── Child Span: "format-response"
```

### Relação de Seguimento (Follows-From)
```
Span A → Span B (Follows-From)
```
- Span B segue logicamente Span A
- Não há relação hierárquica direta

## Atributos e Tags

### Atributos Padrão
- `service.name`: Nome do serviço
- `service.version`: Versão do serviço
- `http.method`: Método HTTP
- `http.status_code`: Código de status HTTP
- `db.system`: Sistema de banco de dados

### Atributos Personalizados
```json
{
  "user.id": "12345",
  "feature.flag": "enabled",
  "cache.hit": true,
  "business.transaction.type": "purchase"
}
```

## Events e Logs

### Events
Events são marcos temporais dentro de um span que capturam informações sobre momentos específicos durante a execução de uma operação.

#### Características dos Events:
- **Timestamp**: Momento exato quando o evento ocorreu
- **Nome**: Identificador descritivo do evento
- **Atributos**: Dados contextuais adicionais
- **Imutabilidade**: Uma vez criados, não podem ser modificados

#### Tipos Comuns de Events:
```json
{
  "timestamp": "2025-01-15T10:30:00.200Z",
  "name": "cache.miss",
  "attributes": {
    "cache.key": "user:12345",
    "cache.type": "redis",
    "cache.ttl": 300
  }
}
```

#### Events de Sistema:
```json
{
  "timestamp": "2025-01-15T10:30:00.300Z",
  "name": "gc.started",
  "attributes": {
    "gc.type": "full",
    "memory.before": "512MB",
    "memory.after": "256MB"
  }
}
```

#### Events de Negócio:
```json
{
  "timestamp": "2025-01-15T10:30:00.400Z",
  "name": "payment.authorized",
  "attributes": {
    "payment.method": "credit_card",
    "payment.amount": 99.99,
    "payment.currency": "USD",
    "payment.gateway": "stripe"
  }
}
```

#### Events vs Logs:
- **Events**: Estruturados, parte do span, focados em marcos específicos
- **Logs**: Podem ser não estruturados, independentes, focados em debugging detalhado

### Status do Span

O status indica o resultado da operação representada pelo span.

#### Estados Possíveis:

##### UNSET (Padrão)
```json
{
  "status": {
    "code": "UNSET"
  }
}
```
- Status padrão quando o span é criado
- Indica que o status não foi explicitamente definido
- Geralmente usado para operações em andamento

##### OK (Sucesso)
```json
{
  "status": {
    "code": "OK",
    "description": "Operation completed successfully"
  }
}
```
- Operação completada com sucesso
- Sem erros detectados
- Resultado dentro das expectativas

##### ERROR (Falha)
```json
{
  "status": {
    "code": "ERROR",
    "description": "Database connection timeout after 30 seconds"
  }
}
```
- Operação falhou
- Deve incluir descrição detalhada do erro
- Usado para rastrear e analisar falhas do sistema

#### Definindo Status Programaticamente:

**Python (OpenTelemetry)**:
```python
from opentelemetry import trace

tracer = trace.get_tracer(__name__)

with tracer.start_as_current_span("database-operation") as span:
    try:
        # Operação do banco de dados
        result = db.execute_query(query)
        span.set_status(trace.Status(trace.StatusCode.OK))
    except DatabaseError as e:
        span.set_status(
            trace.Status(
                trace.StatusCode.ERROR, 
                description=f"Database error: {str(e)}"
            )
        )
        span.record_exception(e)
```

**Java (OpenTelemetry)**:
```java
Span span = tracer.spanBuilder("payment-processing").startSpan();
try (Scope scope = span.makeCurrent()) {
    // Processamento do pagamento
    processPayment();
    span.setStatus(StatusCode.OK);
} catch (PaymentException e) {
    span.setStatus(StatusCode.ERROR, "Payment failed: " + e.getMessage());
    span.recordException(e);
} finally {
    span.end();
}
```

## Correlation ID e Context Propagation

### Correlation ID
Identificador único que conecta eventos relacionados através de diferentes serviços e componentes.

#### Tipos de IDs de Correlação:

##### Trace ID
```
Trace ID: 4bf92f3577b34da6a3ce929d0e0e4736
```
- Identifica todo o trace (requisição completa)
- 128 bits representados em hexadecimal
- Único globalmente
- Propagado através de todos os serviços

##### Span ID
```
Span ID: 00f067aa0ba902b7
```
- Identifica um span específico dentro do trace
- 64 bits representados em hexadecimal
- Único dentro do trace
- Usado para estabelecer relações pai-filho

##### Parent Span ID
```
Parent Span ID: a1b2c3d4e5f67890
```
- Referencia o span pai
- Estabelece hierarquia
- Null para root spans

### Context Propagation

#### Headers HTTP (W3C Trace Context)
```http
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
tracestate: rojo=00f067aa0ba902b7,congo=t61rcWkgMzE
```

**Formato do traceparent:**
```
version-trace_id-parent_id-trace_flags
```

- **version**: `00` (versão atual)
- **trace_id**: 32 caracteres hex (128 bits)
- **parent_id**: 16 caracteres hex (64 bits)
- **trace_flags**: 2 caracteres hex (flags de controle)

#### Baggage
Mecanismo para propagar dados de contexto de negócio através de serviços.

```http
baggage: user-id=12345,tenant-id=abc,session-id=xyz789
```

**Exemplo de uso:**
```python
from opentelemetry import baggage

# Definir baggage
baggage.set_baggage("user.id", "12345")
baggage.set_baggage("feature.experiment", "A")

# Recuperar baggage em outro serviço
user_id = baggage.get_baggage("user.id")
experiment = baggage.get_baggage("feature.experiment")
```

#### Propagação em Diferentes Protocolos:

##### HTTP Headers:
```http
GET /api/users HTTP/1.1
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
tracestate: vendor1=value1,vendor2=value2
baggage: user-id=12345,tenant=production
```

##### gRPC Metadata:
```protobuf
metadata {
  key: "traceparent"
  value: "00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01"
}
```

##### Message Queue Headers (RabbitMQ):
```json
{
  "headers": {
    "traceparent": "00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01",
    "tracestate": "vendor1=value1"
  }
}
```

### Correlação de Logs

#### Structured Logging com Trace Context:
```json
{
  "timestamp": "2025-01-15T10:30:00.123Z",
  "level": "INFO",
  "message": "User authenticated successfully",
  "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
  "span_id": "00f067aa0ba902b7",
  "user_id": "12345",
  "service": "auth-service"
}
```

#### Exemplo de Correlação Completa:

**Request Flow:**
```
1. Frontend (trace_id: abc123)
   └─ Log: "User clicked login button"
   
2. Auth Service (trace_id: abc123, span_id: def456)
   └─ Log: "Validating credentials for user 12345"
   
3. Database (trace_id: abc123, span_id: ghi789, parent: def456)
   └─ Log: "Executing query: SELECT * FROM users WHERE id=12345"
   
4. Auth Service (trace_id: abc123, span_id: def456)
   └─ Log: "Authentication successful for user 12345"
   
5. Frontend (trace_id: abc123)
   └─ Log: "User logged in successfully"
```

#### Query de Correlação (exemplo Elasticsearch):
```json
{
  "query": {
    "bool": {
      "must": [
        { "term": { "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736" } },
        { "range": { "timestamp": { "gte": "2025-01-15T10:30:00", "lte": "2025-01-15T10:35:00" } } }
      ]
    }
  },
  "sort": [
    { "timestamp": { "order": "asc" } }
  ]
}
```

### Melhores Práticas para Correlação:

#### 1. Propagação Consistente:
- Sempre propague context através de boundaries de serviços
- Use bibliotecas padrão (W3C Trace Context)
- Valide propagação em testes

#### 2. Logging Estruturado:
- Inclua sempre trace_id e span_id nos logs
- Use formato JSON para facilitar queries
- Mantenha consistência entre serviços

#### 3. Baggage Consciente:
- Use baggage para dados essenciais de contexto
- Evite dados sensíveis no baggage
- Mantenha baggage pequeno (overhead de rede)

#### 4. Debugging Eficiente:
- Use trace_id para investigar problemas end-to-end
- Correlacione métricas, logs e traces
- Implemente dashboards de correlação

--------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------

## Resumo dos Conceitos Fundamentais

### Trace 🔍
**O que é**: Representa uma jornada completa de uma requisição através do sistema distribuído.

**Características essenciais**:
- **Trace ID único**: Identificador global (128 bits) que conecta todas as operações relacionadas
- **Visibilidade end-to-end**: Mostra o caminho completo da requisição através de múltiplos serviços
- **Duração total**: Tempo desde o início até o fim da operação completa
- **Contexto global**: Container que agrupa todos os spans relacionados

**Exemplo prático**:
```
Trace: "Compra de produto online"
Trace ID: 4bf92f3577b34da6a3ce929d0e0e4736
Duração: 2.5s
Serviços envolvidos: Frontend → API Gateway → Auth → Inventory → Payment → Order
```

### Span 📊
**O que é**: Unidade básica de trabalho que representa uma operação específica dentro de um trace.

**Elementos fundamentais**:
- **Span ID único**: Identificador dentro do trace (64 bits)
- **Timestamps**: Início e fim precisos da operação
- **Hierarquia**: Relação pai-filho com outros spans
- **Metadados**: Atributos, events e status da operação

**Tipos principais**:
```
Root Span     → Ponto de entrada (ex: HTTP request)
Server Span   → Processamento de requisição recebida
Client Span   → Chamada para serviço externo
Internal Span → Processamento interno (ex: cálculos)
```

**Estrutura de um span**:
```json
{
  "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
  "span_id": "00f067aa0ba902b7",
  "parent_span_id": "a1b2c3d4e5f67890",
  "operation_name": "database.query",
  "start_time": "2025-01-15T10:30:00.123Z",
  "end_time": "2025-01-15T10:30:00.456Z",
  "duration_ms": 333,
  "status": "OK"
}
```

### Correlação 🔗
**O que é**: Mecanismo que conecta eventos relacionados através de diferentes sistemas e serviços.

**Componentes principais**:

#### IDs de Correlação:
```
Trace ID  → 4bf92f3577b34da6a3ce929d0e0e4736 (conecta toda a jornada)
Span ID   → 00f067aa0ba902b7 (identifica operação específica)
Parent ID → a1b2c3d4e5f67890 (estabelece hierarquia)
```

#### Propagação de Contexto:
```http
# Header W3C Trace Context
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01

# Baggage para contexto de negócio
baggage: user-id=12345,experiment=feature-A,tenant=production
```

#### Correlação de Logs:
```json
{
  "message": "Payment processed successfully",
  "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
  "span_id": "00f067aa0ba902b7",
  "user_id": "12345",
  "amount": 99.99
}
```

**Benefícios da correlação**:
- **Debugging eficiente**: Rastrear problemas end-to-end
- **Análise de performance**: Identificar gargalos específicos
- **Monitoramento**: Correlacionar métricas, logs e traces
- **Troubleshooting**: Investigar falhas em contexto completo

### Atributos 🏷️
**O que são**: Metadados que enriquecem spans com informações contextuais e técnicas.

#### Categorias de Atributos:

##### Atributos Semânticos (Padrão OpenTelemetry):
```json
{
  "service.name": "payment-service",
  "service.version": "1.2.3",
  "http.method": "POST",
  "http.status_code": 200,
  "http.url": "https://api.example.com/payments",
  "db.system": "postgresql",
  "db.statement": "SELECT * FROM users WHERE id = $1"
}
```

##### Atributos de Negócio:
```json
{
  "user.id": "12345",
  "user.tier": "premium",
  "transaction.type": "purchase",
  "product.category": "electronics",
  "payment.method": "credit_card",
  "feature.flag.checkout_v2": "enabled"
}
```

##### Atributos de Performance:
```json
{
  "cache.hit": true,
  "cache.key": "user:12345:profile",
  "db.rows_affected": 1,
  "queue.size": 42,
  "memory.usage_mb": 256
}
```

#### Melhores Práticas para Atributos:
```python
# ✅ Bom - Atributos estruturados e relevantes
span.set_attributes({
    "user.id": user_id,
    "product.sku": product_sku,
    "inventory.available": stock_count,
    "cache.hit": cache_hit
})

# ❌ Evitar - Dados sensíveis ou muito verbosos
span.set_attributes({
    "user.password": "secret123",  # Dado sensível
    "full_response_body": huge_json  # Muito verboso
})
```

### Integração dos Conceitos 🔄

#### Fluxo Completo:
```
1. REQUEST INICIADA
   └─ Trace criado com ID único
   └─ Root span iniciado

2. PROPAGAÇÃO
   └─ Context propagado via headers HTTP
   └─ Spans filhos criados em cada serviço

3. ENRIQUECIMENTO
   └─ Atributos adicionados em cada operação
   └─ Events marcam momentos importantes
   └─ Status definido (OK/ERROR)

4. CORRELAÇÃO
   └─ Logs incluem trace_id e span_id
   └─ Métricas tagged com contexto
   └─ Debugging facilitado

5. ANÁLISE
   └─ Visualização em waterfall
   └─ Service maps gerados
   └─ Alertas baseados em patterns
```

#### Valor para o Negócio:
- **Visibilidade**: Entendimento completo do sistema
- **Performance**: Identificação de gargalos precisos
- **Confiabilidade**: Detecção rápida de problemas
- **Experiência do usuário**: Otimização baseada em dados reais
- **Custos**: Otimização de recursos e infraestrutura

**Em resumo**: Traces fornecem a visão macro, spans capturam operações específicas, correlação conecta tudo, e atributos fornecem o contexto necessário para análise e debugging eficientes em sistemas distribuídos.