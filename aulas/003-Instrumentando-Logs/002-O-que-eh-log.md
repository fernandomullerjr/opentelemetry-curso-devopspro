
## O que é log

# 📝 Logs no OpenTelemetry: Guia Prático

## 1. O que é um **Log**?

### Definição Simples
Um **log** é um registro textual de um **evento específico** que ocorreu em um momento específico no sistema.

### Características Fundamentais:
```
📅 Timestamp   - Quando aconteceu
💬 Mensagem    - O que aconteceu
📊 Severidade  - Quão importante/sério é (DEBUG, INFO, WARN, ERROR)
🔍 Contexto    - Dados adicionais (métadados)
```

### Exemplos Práticos:
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "ERROR",
  "message": "Falha na conexão com o banco de dados",
  "service": "pedidos-service",
  "user_id": "12345",
  "error_code": "DB_CONN_001"
}
```

## 2. **Logs no OpenTelemetry**

### Evolução: Antes vs OpenTelemetry

| **Tradicional** | **OpenTelemetry** |
|-----------------|-------------------|
| Formato livre | Estrutura padronizada |
| Sem contexto distribuído | Com `Trace ID` e `Span ID` |
| Dificil correlacionar | Facil correlação com traces |
| Múltiplas bibliotecas | API unificada |

### **Estrutura Otel de Logs:**
```go
LogRecord {
  Timestamp: "2024-01-15T10:30:00Z",
  ObservedTimestamp: "2024-01-15T10:30:01Z",
  SeverityText: "ERROR",
  SeverityNumber: 17, // ERROR=17
  Body: "Falha na conexão com BD",
  Attributes: {
    "service.name": "pedidos-service",
    "user.id": "12345",
    "http.status_code": 500
  },
  TraceId: "4bf92f3577b34da6a3ce929d0e0e4736",
  SpanId: "00f067aa0ba902b7"
}
```

### **Integração com Tracing:**
```
Trace (Requisição completa)
├── Span 1: Autenticação
├── Span 2: Busca no BD
│   └── Log: "Query executada: SELECT * FROM users"
└── Span 3: Processamento
    └── Log: "Erro: Campo 'email' inválido"
```

## 3. 📊 **Diferenças: Logs vs Traces vs Metrics**

### **Tabela Comparativa:**

| Aspecto | **Logs** | **Traces** | **Metrics** |
|---------|----------|------------|-------------|
| **Foco** | Eventos específicos | Fluxo de execução | Tendências agregadas |
| **Dados** | Texto + contexto | Hierarquia temporal | Valores numéricos |
| **Granularidade** | Alta (cada evento) | Média (operações) | Baixa (agregados) |
| **Quando usar** | Erros, eventos de negócio | Debug de performance | Monitoramento de saúde |
| **Exemplo** | "Usuário X fez login" | Tempo de requisição API | QPS, uso de CPU |

### **Analogia Médica:**
- **Logs** = Sintomas específicos ("tosse às 10h", "febre 38°C")
- **Traces** = Histórico completo da consulta (triagem → exames → diagnóstico)
- **Metrics** = Sinais vitais constantes (batimentos/minuto, pressão arterial)

## 4. 🎯 **Quando usar cada um?**

### **Use LOGS quando:**
- Registrar erros/exceções
- Eventos de negócio (purchase_completed, user_registered)
- Debug detalhado (valores de variáveis)
- Requer mensagem humana-legível

### **Use TRACES quando:**
- Entender latência entre serviços
- Debug de performance distribuída
- Ver fluxo completo de uma requisição
- Identificar gargalos em microserviços

### **Use METRICS quando:**
- Monitorar SLAs/SLOs
- Alertas (CPU > 80% por 5min)
- Dashboards de tendências
- Auto-scaling decisions

## 5. 🔗 **Correlação no OpenTelemetry**

### **Superpoder do Otel: Conectar tudo!**
```python
# Log com contexto de trace
logger.error("Falha no processamento",
    extra={
        "trace_id": span.get_span_context().trace_id,
        "span_id": span.get_span_context().span_id,
        "attributes": {"order_id": 12345}
    }
)
```

### **Na prática:**
1. **Erro ocorre** → Log é gerado com `Trace ID`
2. **No Jaeger/Grafana** → Busca pelo Trace ID
3. **Resultado**: Vê todos logs + spans + métricas daquela requisição

## 6. 📈 **Padrão de Uso Recomendado**

### **Pirâmide da Observabilidade:**
```
        ⬆ MÉTRICAS (poucos, agregados)
       ╱ ╲
      ╱   ╲
     ╱     ╲
    ╱       ╲
   ╱         ╲
  ╱           ╲
 ╱             ╲
⬆ TRAÇOS        ⬆ LOGS
(alguns por     (muitos, 
 requisição)     detalhados)
```

### **Regra 1-10-100:**
- **1** Trace por requisição crítica
- **10** Métricas principais por serviço
- **100** Logs úteis por operação (não todos DEBUG!)

## 7. ⚠️ **Anti-patterns Comuns**

### ❌ **Não faça:**
```python
# Log sem contexto
logger.info("Processando...")  # O que? Qual usuário?

# Log demais
logger.debug(f"Valor de x: {x}")  # Em loop de 1M iterações

# Usar log onde precisa de métrica
logger.info(f"Tempo resposta: {tempo}ms")  # Use métrica!
```

### ✅ **Faça:**
```python
# Log estruturado com contexto
logger.info("Processando pedido",
    extra={
        "order_id": order.id,
        "user_id": user.id,
        "trace_id": get_current_trace_id()
    }
)

# Logs em nível apropriado
logger.debug("Detalhes internos")  # Desligado em produção
logger.error("Erro irrecoverável", exc_info=True)
```

## 8. 🚀 **Quick Start com Otel Logs**

```python
from opentelemetry import trace
from opentelemetry.sdk._logs import LoggerProvider
import logging

# Setup
logger_provider = LoggerProvider()
logging.basicConfig(level=logging.INFO)

# Log com contexto Otel
tracer = trace.get_tracer(__name__)
with tracer.start_as_current_span("process-order") as span:
    # Log automaticamente correlacionado
    logging.getLogger().info(
        "Pedido processado",
        extra={
            "attributes": {"order_id": "123"},
            "trace_id": format(span.get_span_context().trace_id, '032x'),
            "span_id": format(span.get_span_context().span_id, '016x')
        }
    )
```

## 9. 📋 **Checklist de Implementação**

- [ ] Logs estruturados (JSON)
- [ ] Contexto obrigatório (service, trace_id)
- [ ] Níveis apropriados (INFO, ERROR)
- [ ] Sem dados sensíveis em logs
- [ ] Exportação para coletor Otel
- [ ] Logs correlacionados com traces
- [ ] Retenção definida (30-90 dias)

## 10. 🔍 **Ferramentas Recomendadas**

- **Visualização**: Grafana Loki, Elasticsearch
- **Análise**: Grafana, Kibana
- **Coleta**: Otel Collector + filelog receiver
- **Storage**: S3/GCS para arquivamento

---

**Resumo Final:**
- **Logs** = "O QUÊ aconteceu" (eventos discretos)
- **Traces** = "COMO aconteceu" (relacionamentos temporais)
- **Metrics** = "QUANTO acontece" (agregados numéricos)

**No OpenTelemetry:** Todos correlacionados via `Trace ID` para observabilidade completa!