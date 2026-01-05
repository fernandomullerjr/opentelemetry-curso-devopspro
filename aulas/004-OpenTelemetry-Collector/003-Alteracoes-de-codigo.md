
## Alterações de código


- Variáveis foram centralizadas no arquivo config.py:

~~~~py
import os
from dotenv import load_dotenv

# ================================
#  CONFIGURAÇÃO DA APLICAÇÃO
# ================================

load_dotenv()

APP_NAME = os.getenv("APP_NAME", "app-a")
APP_URL_DESTINO = os.getenv("APP_URL_DESTINO", "")

# Simulação de problemas
APP_ERRORS = int(os.getenv("APP_ERRORS", "0"))  # Porcentagem de erro (0 a 100)
APP_LATENCY = int(os.getenv("APP_LATENCY", "0"))  # Tempo máximo de atraso (em ms)

OTLP_ENDPOINT = os.getenv("OTLP_ENDPOINT", "") 
~~~~





- Devido uso do OTLP é necessário ter este export no arquivo logs.py:

~~~~py
from opentelemetry.exporter.otlp.proto.http._log_exporter import OTLPLogExporter
~~~~

obs:
como está em desenvolvimento, tem o underline no nome dele

-  Configura o exportador de logs para OTLP:

~~~~py
# Configura o exportador de logs para OTLP
otlp_exporter = OTLPLogExporter(
    endpoint=f"{config.OTLP_ENDPOINT}/v1/logs" 
)
logger_provider.add_log_record_processor(
    SimpleLogRecordProcessor(otlp_exporter)
)
~~~~



- No arquivo metrics.py

~~~~py
from opentelemetry.sdk.metrics.export import PeriodicExportingMetricReader
from opentelemetry.exporter.otlp.proto.http.metric_exporter import OTLPMetricExporter

otlp_exporter = OTLPMetricExporter(
    endpoint=f"{config.OTLP_ENDPOINT}/v1/metrics"
)

otlp_reader = PeriodicExportingMetricReader(
    exporter=otlp_exporter,
    export_interval_millis=1000 # Exporta a cada 1 segundo
)
~~~~



