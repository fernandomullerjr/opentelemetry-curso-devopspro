
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