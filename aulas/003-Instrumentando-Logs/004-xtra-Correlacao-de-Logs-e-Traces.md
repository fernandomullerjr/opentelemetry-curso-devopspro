
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


