

## Adicionado métricas Histogram

- Código lá no metrics.py:

~~~~py
# =============================================================================
# HISTOGRAM (HISTOGRAMA) - MÉTRICA DE DISTRIBUIÇÃO
# =============================================================================
# Histogram: Agrupa observações em buckets (caixas) baseado em valores
# Ideal para medir latência, tamanho de requisições, etc.
# Permite analisar percentis (ex: 95% das requisições respondem em menos de X segundos)
response_time_histogram = meter.create_histogram(
    name="app_response_time_seconds",
    description="Tempo de resposta das requisições em segundos",
    unit="s",                           # Unidade: segundos
    explicit_bucket_boundaries_advisory=[  # Limites dos buckets em segundos
        0.005,  # 5ms
        0.01,   # 10ms
        0.025,  # 25ms
        0.05,   # 50ms
        0.1,    # 100ms
        0.25,   # 250ms
        0.5     # 500ms
    ]
)

~~~~


- Lá no app.py é necessário importar o "response_time_histogram":

~~~~py
from otel.metrics import requests_counter, active_requests_gauge, response_time_histogram
~~~~


- Nó código do process
pegar o tempo e hora atual

~~~~py
start_time = time.time()
~~~~

e ao final pegar o tempo decorrido:

~~~~py
elapsed_time = time.time() - start_time
~~~~

para obter o tempo de duração do processamento