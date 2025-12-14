

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



    start_time = time.time()
    elapsed_time = time.time() - start_time
    response_time_histogram.record(elapsed_time, {"app": config.APP_NAME, "endpoint": "/"})


# TYPE app_response_time_seconds histogram
app_response_time_seconds_bucket{app="app-a",endpoint="/",le="0.005"} 10.0
app_response_time_seconds_bucket{app="app-a",endpoint="/",le="0.01"} 10.0
app_response_time_seconds_bucket{app="app-a",endpoint="/",le="0.025"} 10.0
app_response_time_seconds_bucket{app="app-a",endpoint="/",le="0.05"} 10.0
app_response_time_seconds_bucket{app="app-a",endpoint="/",le="0.1"} 10.0
app_response_time_seconds_bucket{app="app-a",endpoint="/",le="0.25"} 10.0
app_response_time_seconds_bucket{app="app-a",endpoint="/",le="0.5"} 10.0
app_response_time_seconds_bucket{app="app-a",endpoint="/",le="+Inf"} 10.0



- Simulando no path /process com APP_LATENCY=250


app_response_time_seconds_bucket{app="app-a",endpoint="/process",le="0.005"} 0.0
app_response_time_seconds_bucket{app="app-a",endpoint="/process",le="0.01"} 0.0
app_response_time_seconds_bucket{app="app-a",endpoint="/process",le="0.025"} 0.0
app_response_time_seconds_bucket{app="app-a",endpoint="/process",le="0.05"} 0.0
app_response_time_seconds_bucket{app="app-a",endpoint="/process",le="0.1"} 0.0
app_response_time_seconds_bucket{app="app-a",endpoint="/process",le="0.25"} 0.0
app_response_time_seconds_bucket{app="app-a",endpoint="/process",le="0.5"} 5.0
app_response_time_seconds_bucket{app="app-a",endpoint="/process",le="+Inf"} 5.0