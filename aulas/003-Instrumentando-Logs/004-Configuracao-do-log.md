
## Configuração do Log

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


## Dia 31/12/2025

- Sobre o Resource

~~~~py
# Cria um recurso OpenTelemetry com informações do serviço
# Isso ajuda a identificar a origem dos logs
resource = Resource.create({
    SERVICE_NAME: config.APP_NAME,  # Nome do serviço
    SERVICE_VERSION: "1.0.0"  # Versão do serviço
})
~~~~


- Aqui foi hard-coded, mas pode ser obtido de uma variável, etc
importante é ter a informação!




- Sobre o LoggerProvider:

~~~~py
# Inicializa o provedor de logs do OpenTelemetry
# Este é o componente principal que gerencia os logs
logger_provider = LoggerProvider(resource=resource)
~~~~

Ele cria o “provedor” de logs do OpenTelemetry para sua aplicação:

- resource define metadados do serviço (service.name e service.version) que serão anexados a cada log.
- LoggerProvider(resource=resource) inicializa o pipeline de logs com esses metadados.
- Em seguida (nas linhas abaixo), você adiciona processadores que enviam os logs para o console e para o OTLP, e registra esse provider como global com set_logger_provider.
- O LoggingHandler usa esse provider para transformar logs do logging padrão (logger.info, etc.) em LogRecords OpenTelemetry com o resource, que são então exportados pelos processadores configurados.


- Sobre o ConsoleLogExporter:

~~~~py
# Configura o exportador de logs para console
# Isso permite que os logs sejam exibidos no terminal
console_exporter = ConsoleLogExporter()
logger_provider.add_log_record_processor(
    SimpleLogRecordProcessor(console_exporter)
)

# Define o provedor de logs como global
# Isso permite que outros componentes da aplicação usem o mesmo provedor
set_logger_provider(logger_provider)
~~~~

Ele conecta o provedor de logs ao console:

- ConsoleLogExporter: formata e escreve cada LogRecord no stdout (terminal), útil para debug.
- add_log_record_processor(SimpleLogRecordProcessor(console_exporter)): registra um processador que envia cada log imediatamente ao exporter, de forma síncrona (sem buffer ou retentativas).

Resultado: sempre que seu logger emitir um log, ele aparecerá no terminal.



### IMPORTANTE
- Lembrar sobre a importancia de criar o bloco do Resource, pois ajuda a identificar a origem dos logs.
