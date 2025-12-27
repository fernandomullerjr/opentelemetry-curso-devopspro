
## Enriquecendo seus Traces e Spans

- Necessário importar:

~~~~py
from opentelemetry.semconv.attributes.http_attributes import (
    HTTP_ROUTE,
    HTTP_REQUEST_METHOD,
    HttpRequestMethodValues
)
~~~~


- Setando atributos no Span principal:

~~~~py
    with tracer.start_as_current_span("process-request", context=context) as main_span:

        original_payload = payload.copy()
        original_payload.append(config.APP_NAME)

        main_span.set_attribute(HTTP_ROUTE, "/process")
        main_span.set_attribute(HTTP_REQUEST_METHOD, HttpRequestMethodValues.POST)
        main_span.set_attribute("app.name", config.APP_NAME)
        main_span.set_attribute("payload.original", str(payload))
        main_span.set_attribute("payload.modified", str(original_payload))
~~~~

- Setando atributos no Span principal:

~~~~py

                        child_span.set_attribute("net.peer.name", url)
                        child_span.set_attribute("destination.url", url)

                        child_span.add_event("Requisição externa bem-sucedida", {"url": url})

                        resp = requests.post(
                            f"{url}/process",
                            json=original_payload,
                            headers=headers,
                            timeout=5
                        )

                        child_span.set_attribute("http.status_code", resp.status_code)

~~~~


É importante botar na documentação do projeto o que cada chave faz, exemplo, trazer detalhes sobre:
("app.name", config.APP_NAME)


- Efetuando requisições de teste, usando o teste_request.http

- Olhando na UI do Jaeger, é possível expandir e ver os detalhes do Span contendo os atributos:

process-request

    Service:app-a
    Duration:319.27ms
    Start Time:0μs

Tags
    app.name	
    app-a
    http.route	
    /process
    otel.scope.name	
    app-a
    otel.status_code	
    OK
    payload.modified	
    ['app-a']
    payload.original	
    []
    span.kind	
    internal



- Exemplo contendo informações sobre payload original e o modificado:

process-request

    Service:app-b
    Duration:202.37ms
    Start Time:9.16ms

Tags
    app.name	
    app-b
    http.route	
    /process
    otel.scope.name	
    app-b
    otel.status_code	
    OK
    payload.modified	
    ['app-a', 'app-b']
    payload.original	
    ['app-a']
    span.kind	
    internal




- Adicionando informações ao Span
- Importante cuidar para não adicionar algo que fere o Compliance ou informações sigilosas
- Também é importante não adicionar informações desnecessárias:

~~~~py
        main_span.add_event("Início do processamento", 
                {"payload_tamanho": f"{sys.getsizeof(payload)} bytes"}
        )
~~~~



- Verificando Span contendo erro:

process-request

    Service:app-a
    Duration:98.82ms
    Start Time:0μs

Tags
    app.name	
    app-a
    error	
    true
    http.route	
    /process
    http.status_code	
    500
    otel.scope.name	
    app-a
    otel.status_code	
    ERROR
    payload.modified	
    ['app-a']
    payload.original	
    []
    span.kind	
    internal



- É possível ver que ocorre incremento em cada payload conforme vão sendo incrementados o tamanho cresce:

app-a

 Logs (1)
124μs
event	
Início do processamento
payload_tamanho	
56 bytes

app-b

 Logs (1)
70.26ms
event	
Início do processamento
payload_tamanho	
64 bytes

app-c

 Logs (1)
107.27ms
event	
Início do processamento
payload_tamanho	
72 bytes



- Para enriquecer os erros, é possível fazer algo assim:

~~~~py

        # Simulação de erro com base na porcentagem definida
        if random.randint(1, 100) <= config.APP_ERRORS:
            response.status_code = status.HTTP_500_INTERNAL_SERVER_ERROR

            error_msg = f"Erro simulado em {config.APP_NAME}"
            
            # Log estruturado do erro
            logger.error(
                "Erro simulado durante processamento",
                extra={
                    **log_context,
                    "error_message": error_msg,
                    "error_type": "simulated_error",
                    "error_percentage": config.APP_ERRORS,
                    "payload": payload.copy()
                }
            )

            logger.critical(
                "Erro fatal simulado durante processamento",
                extra={
                    **log_context,
                    "error_message": error_msg,
                    "error_type": "simulated_error",
                    "error_percentage": config.APP_ERRORS,
                    "payload": payload.copy()
                }
            )

            main_span.record_exception(Exception(error_msg))
            main_span.set_status(Status(StatusCode.ERROR))
            main_span.set_attribute("http.status_code", status.HTTP_500_INTERNAL_SERVER_ERROR)

            return {"error": error_msg}

~~~~




como funciona o " main_span.set_status(Status(StatusCode.OK))" neste código?

## Importante reforçar

### 3. Diferença entre Status HTTP e Status do Span

É importante não confundir o status da resposta HTTP com o status do Span do OpenTelemetry:

* **Response Status (FastAPI):** `response.status_code = 200` diz ao **navegador/cliente** que deu tudo certo.
* **Span Status (OTEL):** `main_span.set_status(StatusCode.OK)` diz ao **backend de observabilidade** que a lógica interna daquela unidade de trabalho foi finalizada corretamente.