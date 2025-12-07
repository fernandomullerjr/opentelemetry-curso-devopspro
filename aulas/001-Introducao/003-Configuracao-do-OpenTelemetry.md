

## Configuração do OpenTelemetry


- Finalizar aula "Configuração do OpenTelemetry"
em 04:53min


- Instalação de pacotes e dependencias
pip install
usando os requirements
no video é via python, pip install
como é via Docker o ambiente, no Dockerfile já tem todo o pip install necessário


- Para organizar o código
criar um arquivo Python para as métricas
facilita a manutenção
criar pasta chamada "otel"
arquivo metrics.py:
/home/fernando/cursos/opentelemetry/opentelemetry-curso-devopspro/app-telemetria/src/otel/metrics.py


# Importações necessárias para trabalhar com métricas OpenTelemetry
from opentelemetry import metrics
from opentelemetry.sdk.metrics import MeterProvider
from opentelemetry.exporter.prometheus import PrometheusMetricReader

# PrometheusMetricReader: Responsável por expor as métricas no formato que o Prometheus entende
# O Prometheus é uma ferramenta de monitoramento que coleta e armazena métricas
prometheus_reader = PrometheusMetricReader()