

## Adicionando métricas Gauge

- É possível fazer nos 2 formatos:
Sincrono
Assincrono


- No arquivo metrics.py:

~~~~py
# =============================================================================
# GAUGE (MEDIDOR) - MÉTRICA DE VALOR ATUAL
# =============================================================================
# Gauge: Representa um valor atual que pode subir ou descer
# Ex: temperatura, uso de memória, número de conexões ativas
active_requests_gauge = meter.create_gauge(
    name="app_active_requests",
    description="Número de requisições ativas",
    unit="1",
)

~~~~


- Lá no arquivo app.py é necessário adicionar o "active_requests_gauge":

~~~~py
from otel.metrics import requests_counter, active_requests_gauge, response_time_histogram
~~~~


- E exemplo de uso dele no path root:

~~~~py
@app.get("/")
def read_root():
    # Log estruturado para endpoint raiz
    logger.info(
        "Endpoint raiz acessado",
        extra={
            "service_name": config.APP_NAME,
            "endpoint": "/",
            "operation": "health_check"
        }
    )
    
    active_requests_gauge.set(1, {"app": config.APP_NAME})
    requests_counter.add(1, {"app": config.APP_NAME, "endpoint": "/"})
    start_time = time.time()
    elapsed_time = time.time() - start_time
    response_time_histogram.record(elapsed_time, {"app": config.APP_NAME, "endpoint": "/"})
    return {"message": f"Esse é o serviço {config.APP_NAME}"}
~~~~


- Importante
no Gauge eu seto um valor
no Counter eu adiciono um valor
conforme:

~~~~py
    active_requests_gauge.set(1, {"app": config.APP_NAME})
    requests_counter.add(1, {"app": config.APP_NAME, "endpoint": "/"})
~~~~



### Assincrono

~~~~py
# =============================================================================
# OBSERVABLE GAUGE (MEDIDOR OBSERVÁVEL) - GAUGE COM CALLBACK
# =============================================================================
# Observable Gauge: Valor atual calculado dinamicamente
# Perfeito para métricas do sistema que mudam constantemente

# Obtém referência ao processo atual para monitorar recursos
process = psutil.Process()

def get_memory_usage(options: CallbackOptions) -> Iterable[Observation]:
    """
    Callback function que monitora o uso de memória do processo
    Demonstra como coletar métricas do sistema operacional
    """
    memory_usage = process.memory_percent()  # Uso de memória em porcentagem
    yield Observation(
        memory_usage,                    # Valor da métrica (porcentagem)
        {"service": APP_NAME}           # Atributos para identificação
    )

# Criação do observable gauge para monitoramento de memória
memory_gauge = meter.create_observable_gauge(
    name="app_memory_usage",
    description="Uso de memória do processo",
    callbacks=[get_memory_usage]        # Função que calcula o uso de memória
)
~~~~


- Lá no endpoint http://localhost:8000/metrics
<http://localhost:8000/metrics>
a métrica gerada:

# TYPE app_memory_usage gauge
app_memory_usage{service="app-a"} 0.44093307919614944