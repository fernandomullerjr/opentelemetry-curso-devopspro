# 📊 **Níveis de Logs no OpenTelemetry: Guia Completo**

## 1. 🎯 **Visão Geral dos Níveis**

### **Hierarquia de Severidade (mais baixo → mais alto):**

```
TRACE/VERBOSE/DEBUG (1-4)   ← Mais detalhado, desenvolvimento
INFO (5-8)                   ← Informações normais de operação
WARN/WARNING (9-12)          ← Situações anormais, mas não críticas
ERROR (13-16)                ← Falhas em funcionalidades específicas
FATAL/CRITICAL (17-20)       ← Falhas que impedem continuidade
```

## 2. 📋 **Tabela de Mapeamento Detalhada**

| **SeverityNumber** | **SeverityText** | **Quando Usar** | **Exemplo** | **Em Produção?** |
|-------------------|------------------|-----------------|-------------|------------------|
| **1** | TRACE | Detalhes extremamente granulares | `Valor de loop: i=42, array_len=100` | ❌ Não |
| **5** | DEBUG | Informações para debugging | `Query SQL: SELECT * FROM users WHERE id=?` | ⚠️ Limitado |
| **9** | INFO | Confirmação de operações normais | `Usuário autenticado: user_id=123` | ✅ Sim |
| **13** | WARN | Situações potencialmente problemáticas | `Cache miss para chave: user_123` | ✅ Sim |
| **17** | ERROR | Falhas em operações específicas | `Falha ao processar pagamento: order_id=456` | ✅ Sim |
| **21** | FATAL | Falhas críticas do sistema | `Database connection lost` | ✅ Sim |

## 3. 🔢 **SeverityNumber vs SeverityText**

### **OpenTelemetry usa dois sistemas:**

```go
// Sistema Numérico (padronizado)
type SeverityNumber int

const (
    SeverityNumberTRACE  SeverityNumber = 1
    SeverityNumberDEBUG  SeverityNumber = 5
    SeverityNumberINFO   SeverityNumber = 9
    SeverityNumberWARN   SeverityNumber = 13
    SeverityNumberERROR  SeverityNumber = 17
    SeverityNumberFATAL  SeverityNumber = 21
)

// Sistema Textual (traduzível)
type SeverityText string

const (
    SeverityTextTRACE  SeverityText = "TRACE"
    SeverityTextDEBUG  SeverityText = "DEBUG"
    // ... etc
)
```

### **Conversão Automática:**
```python
# Otel faz conversão automática
log_record = LogRecord(
    severity_number=17,           # ERROR numeric
    severity_text="ERROR",        # ERROR textual
    body="Failed to connect to DB"
)
```

## 4. 🎚️ **Configuração por Ambiente**

### **Development/Local:**
```yaml
logging:
  level: "DEBUG"  # Vê tudo
  export_levels: ["DEBUG", "INFO", "WARN", "ERROR", "FATAL"]
```

### **Staging/QA:**
```yaml
logging:
  level: "INFO"   # Sem DEBUG
  export_levels: ["INFO", "WARN", "ERROR", "FATAL"]
```

### **Production:**
```yaml
logging:
  level: "WARN"   # Apenas problemas e erros
  export_levels: ["WARN", "ERROR", "FATAL"]
  sampled_debug: 1%  # DEBUG amostrado para troubleshooting
```

## 5. 📈 **Filtros Dinâmicos no Otel Collector**

```yaml
# otel-collector-config.yaml
processors:
  filter:
    logs:
      log_record:
        # Apenas logs ERROR e FATAL vão para o alerting
        include:
          match_type: regexp
          severity_texts: ["ERROR", "FATAL"]
        
        # DEBUG apenas para dev environments
        exclude:
          match_type: strict
          severity_text: "DEBUG"
          attributes:
            - key: environment
              value: production
```

## 6. 🏷️ **Atributos Recomendados por Nível**

### **Cada nível deve ter contextos específicos:**

```json
{
  "DEBUG": {
    "timestamp": "2024-01-15T10:30:00Z",
    "severity": "DEBUG",
    "message": "Calculating user score",
    "attributes": {
      "service": "scoring-service",
      "function": "calculate_score",
      "input_values": {"age": 30, "purchases": 15},
      "trace_id": "abc123"
    }
  },
  
  "ERROR": {
    "timestamp": "2024-01-15T10:31:00Z",
    "severity": "ERROR",
    "message": "Failed to charge credit card",
    "attributes": {
      "service": "payment-service",
      "error_code": "PAYMENT_001",
      "user_id": "12345",
      "order_id": "ORD67890",
      "payment_gateway": "stripe",
      "trace_id": "abc123",
      "span_id": "def456",
      "retry_count": 3
    }
  }
}
```

## 7. ⚡ **Performance Considerations**

### **Custo de cada nível (ordem crescente):**

```
TRACE/DEBUG → INFO → WARN → ERROR → FATAL
     ↑                            ↑
  Mais caro                   Mais barato
  (volume alto)              (volume baixo)
```

### **Regras de Otimização:**
```python
# ❌ EVITE (custo alto, pouco valor)
logger.debug(f"User object: {user.__dict__}")  # Serializa objeto inteiro

# ✅ PREFIRA (custo baixo, alto valor)
if logger.isEnabledFor(logging.DEBUG):
    logger.debug("User auth attempt: %s", user.id)  # Avaliação lazy
```

## 8. 🔄 **Mapeamento com Bibliotecas Existentes**

### **Python logging → Otel:**
```python
import logging
from opentelemetry.sdk._logs import LoggingHandler

# Configuração
logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)

# Mapeamento automático
handler = LoggingHandler()
logger.addHandler(handler)

# Uso
logger.debug("Debug message")    # → SeverityNumber=5
logger.info("Info message")      # → SeverityNumber=9
logger.warning("Warning message") # → SeverityNumber=13
logger.error("Error message")    # → SeverityNumber=17
logger.critical("Critical msg")  # → SeverityNumber=21
```

### **Java SLF4J → Otel:**
```java
// Mapeamento automático via Otel appender
TRACE → SeverityNumber.TRACE
DEBUG → SeverityNumber.DEBUG
INFO  → SeverityNumber.INFO
WARN  → SeverityNumber.WARN
ERROR → SeverityNumber.ERROR
```

## 9. 🚨 **Alertas Baseados em Níveis**

### **Configuração de Alerting:**
```yaml
# Grafana/Prometheus alert rules
groups:
  - name: logs_alerts
    rules:
      - alert: HighErrorRate
        expr: |
          sum(rate(log_entries{severity="ERROR"}[5m])) 
          / 
          sum(rate(log_entries{severity!~"DEBUG|TRACE"}[5m])) > 0.05
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "Alta taxa de erros no serviço {{ $labels.service }}"
```

## 10. 📊 **Dashboard por Nível (Exemplo Grafana)**

```sql
-- Logs por nível (últimas 24h)
SELECT
  severity_text,
  COUNT(*) as log_count,
  COUNT(DISTINCT trace_id) as unique_traces
FROM logs
WHERE time > now() - 24h
GROUP BY severity_text
ORDER BY 
  CASE severity_text
    WHEN 'FATAL' THEN 1
    WHEN 'ERROR' THEN 2
    WHEN 'WARN' THEN 3
    WHEN 'INFO' THEN 4
    WHEN 'DEBUG' THEN 5
    WHEN 'TRACE' THEN 6
  END
```

## 11. 🎯 **Boas Práticas por Nível**

### **DEBUG/TRACE:**
- Use apenas para troubleshooting
- Inclua contexto suficiente para reproduzir
- Desative em produção ou use sampling
- Exemplo bom: `DEBUG: Calculated score=85 for user=123 [input={age:30}]`

### **INFO:**
- Eventos significativos de negócio
- Mudanças de estado importantes
- Exemplo: `INFO: Order status changed: pending→paid [order=456]`

### **WARN:**
- Situações recuperáveis mas anormais
- Performance degradada
- Exemplo: `WARN: High response time: 2.5s [threshold=1s, endpoint=/api/users]`

### **ERROR:**
- Falhas que requerem atenção
- Inclua sempre: error_code, recovery_steps, contexto
- Exemplo: `ERROR: Payment failed [code=PAY_001, user=123, order=456, retry=3]`

### **FATAL/CRITICAL:**
- Apenas para falhas irrecoveráveis
- Deve acionar pager/oncall
- Exemplo: `FATAL: Database cluster unreachable [shard=primary, downtime=5min]`

## 12. 🧪 **Exemplo Completo em Código**

```python
from opentelemetry import trace
from opentelemetry._logs import set_logger_provider
from opentelemetry.sdk._logs import LoggerProvider, LoggingHandler
import logging

# Setup
logger_provider = LoggerProvider()
set_logger_provider(logger_provider)
handler = LoggingHandler(level=logging.NOTSET)
logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)
logger.addHandler(handler)

# Exemplos de uso correto
def process_order(order_id: str):
    tracer = trace.get_tracer(__name__)
    
    with tracer.start_as_current_span("process-order") as span:
        # DEBUG - detalhes internos
        logger.debug("Starting order processing", 
            extra={"order_id": order_id, "step": "validation"})
        
        # INFO - eventos de negócio
        logger.info("Order validated successfully",
            extra={"order_id": order_id, "validation_time_ms": 150})
        
        try:
            # Operação que pode falhar
            charge_payment(order_id)
            
        except PaymentError as e:
            # ERROR - falha específica
            logger.error("Payment processing failed",
                extra={
                    "order_id": order_id,
                    "error_code": e.code,
                    "retry_count": 3,
                    "trace_id": format(span.get_span_context().trace_id, '032x')
                },
                exc_info=True  # Inclui stack trace
            )
            
        except DatabaseConnectionError as e:
            # FATAL - falha crítica
            logger.critical("Database connection lost",
                extra={
                    "service": "order-service",
                    "error": str(e),
                    "impact": "all_operations_halted"
                })
            raise
```

## 13. 📋 **Checklist de Implementação**

- [ ] Definir política de níveis por ambiente
- [ ] Configurar filtros no collector
- [ ] Mapear biblioteca de logging para Otel
- [ ] Incluir contexto mínimo (service, trace_id)
- [ ] Configurar alertas para ERROR/FATAL
- [ ] Definir retenção por nível (DEBUG: 7 dias, ERROR: 90 dias)
- [ ] Implementar log sampling para DEBUG/TRACE
- [ ] Documentar convenções da equipe

## 14. 🔍 **Debug Avançado: Conditional Logging**

```python
# Log condicional para performance
if logger.isEnabledFor(logging.DEBUG):
    # Cálculo custoso só se DEBUG ativo
    debug_info = calculate_debug_info(request)
    logger.debug("Request details: %s", debug_info)

# Structured logging com severidade dinâmica
def log_with_context(level: str, message: str, **kwargs):
    log_level = getattr(logging, level.upper())
    if logger.isEnabledFor(log_level):
        logger.log(
            log_level,
            message,
            extra={
                "attributes": kwargs,
                "trace_id": get_current_trace_id()
            }
        )
```

---

**Regra de Ouro:** 
> "Log como se quem for litar estivesse de pijama às 3AM com o pager na mão - seja claro, conciso e acionável."

**Níveis são ferramentas, não regras.** Ajuste baseado em:
- Volume de logs suportado
- Necessidades da equipe
- Custos de storage/processamento
- Requisitos de compliance/auditoria