

## Adicionando métricas Counter

- No arquivo metrics.py, ajustar para que ele faça uma contagem

~~~~py
# =============================================================================
# COUNTER (CONTADOR) - MÉTRICA CUMULATIVA
# =============================================================================
# Counter: Só aumenta, nunca diminui (ex: número total de requisições)
# Ideal para contar eventos que acontecem ao longo do tempo
requests_counter = meter.create_counter(
    name="app_requests_total",           # Nome da métrica (padrão: nome_total)
    description="Número de requisições processadas",  # Descrição humana
    unit="1",                            # Unidade de medida (1 = contagem)
)
~~~~


- Esta informações contidas no campo "description", por exemplo, vem dispostas no path do http://localhost:8000/metrics:

~~~~
# HELP app_requests_total Número de requisições processadas
# TYPE app_requests_total counter
app_requests_total{app="app-a",endpoint="/"} 4.0
~~~~

ajustando a identificar o que faz aquela métrica
no exemplo acima eu tinha acessado 4 vezes o app-a, para gerar métricas


- Lá no app.py é necessário importar:
from otel.metrics import requests_counter

- E adicionar no código o trecho para contabilizar +1 e adicionar labels:

~~~~py
@app.get("/")
def read_root():
    requests_counter.add(1, {"app": config.APP_NAME, "endpoint": "/"})
    return {"message": f"Esse é o serviço {config.APP_NAME}"}
~~~~


- Também é possível fazer o mesmo para o path /process, instrumentando o contador:

~~~~py
requests_counter.add(1, {"app": config.APP_NAME, "endpoint": "/process"})
~~~~