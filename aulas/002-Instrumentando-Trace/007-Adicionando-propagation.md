

## Adicionando Propagation


from opentelemetry.trace.propagation.tracecontext import TraceContextTextMapPropagator


Ele é responsável por passar o Correlation ID



tracing.py

~~~~py
# =============================================================================
# PROPAGADOR DE CONTEXTO
# =============================================================================
# TraceContextTextMapPropagator: Responsável por propagar o contexto de trace
# entre diferentes serviços através de headers HTTP ou outros meios
# Permite que uma requisição mantenha seu contexto de trace ao atravessar
# múltiplos serviços (ex: API Gateway -> Serviço A -> Serviço B)
propagator = TraceContextTextMapPropagator()
~~~~






app.py

~~~~py
from otel.tracing import tracer, propagator

    context = propagator.extract(request.headers)



                        headers = {}
                        propagator.inject(headers)



~~~~

O arquivo mostra a implementação de **propagação de contexto** usando `TraceContextTextMapPropagator` para manter o **Correlation ID** entre serviços. No endpoint `/process`, o contexto é **extraído** dos headers da requisição de entrada com `propagator.extract(request.headers)` e **injetado** nos headers das requisições downstream com `propagator.inject(headers)`. Isso permite que o mesmo trace ID seja mantido através de toda a cadeia de serviços, facilitando a correlação de logs e traces distribuídos. 
O contexto extraído é usado para iniciar spans filhos que herdam o trace ID original, criando uma visão unificada do fluxo da requisição.