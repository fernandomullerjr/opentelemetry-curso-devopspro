
## Configuração do OpenTelemetry


Provider = Tipo de coleta
Exporter = Envio de dados



- Arquivo onde iremos configurar:
opentelemetry-curso-devopspro/app-telemetria/src/otel/logs.py

~~~~py
# Importação das bibliotecas necessárias
import logging
import config
from opentelemetry.sdk._logs import LoggerProvider, LoggingHandler
from opentelemetry.sdk._logs.export import ConsoleLogExporter, SimpleLogRecordProcessor
from opentelemetry.sdk.resources import Resource
from opentelemetry.semconv.attributes.service_attributes import (
    SERVICE_NAME,
    SERVICE_VERSION
)
from opentelemetry._logs import set_logger_provider
from opentelemetry.exporter.otlp.proto.http._log_exporter import OTLPLogExporter
~~~~


- Observação:
o underscore diz que aquela biblioteca está em construção / desenvolvimento

## CONTINUA EM:
08:11min