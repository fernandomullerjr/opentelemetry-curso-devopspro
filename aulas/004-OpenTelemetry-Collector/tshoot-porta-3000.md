


### PROBLEMA

Ao tentar acessar
http://localhost:3000/

Traz referencia ao OpenWebui, que já teve container deletado e sujeiras também.

Retorna a mensagem:
"Backend Open WebUI necessário
Ops! Você está usando um método não suportado (somente frontend). Por favor, sirva a WebUI a partir do backend.

Veja readme.md para instruções ou junte-se ao nosso Discord para ajudar."


- Verificando os containers:

> docker ps
CONTAINER ID   IMAGE                                          COMMAND                  CREATED         STATUS         PORTS                                                                                                                                     NAMES
478996bbc71a   fabriciosveronez/app-telemetria:v1             "uvicorn app:app --h…"   2 minutes ago   Up 2 minutes   0.0.0.0:8000->8000/tcp                                                                                                                    app-telemetria-app-a-1
4507ffd2ab54   app-telemetria-app-c                           "uvicorn app:app --h…"   2 minutes ago   Up 2 minutes   0.0.0.0:8002->8000/tcp                                                                                                                    app-telemetria-app-c-1
2a47d08a23e0   app-telemetria-app-b                           "uvicorn app:app --h…"   2 minutes ago   Up 2 minutes   0.0.0.0:8001->8000/tcp                                                                                                                    app-telemetria-app-b-1
ca1a8a28488f   otel/opentelemetry-collector-contrib:0.128.0   "/otelcol-contrib --…"   2 minutes ago   Up 2 minutes   0.0.0.0:4317-4318->4317-4318/tcp, 0.0.0.0:8889->8889/tcp, 55678-55679/tcp                                                                 app-telemetria-otel-collector-1
6a8340d13af2   grafana/grafana:latest                         "sh -euc 'mkdir -p /…"   2 minutes ago   Up 2 minutes   0.0.0.0:3000->3000/tcp                                                                                                                    app-telemetria-grafana-1
45ece07584c4   grafana/loki:latest                            "/usr/bin/loki -conf…"   2 minutes ago   Up 2 minutes   0.0.0.0:3100->3100/tcp                                                                                                                    app-telemetria-loki-1
a62466a96cf2   prom/prometheus:latest                         "/bin/prometheus --c…"   2 minutes ago   Up 2 minutes   0.0.0.0:9090->9090/tcp                                                                                                                    prometheus
9b9a1b43f941   jaegertracing/all-in-one:latest                "/go/bin/all-in-one-…"   2 minutes ago   Up 2 minutes   4317-4318/tcp, 0.0.0.0:14250->14250/tcp, 0.0.0.0:14268->14268/tcp, 9411/tcp, 0.0.0.0:6831-6832->6831-6832/udp, 0.0.0.0:16686->16686/tcp   jaeger
> curl -I http://127.0.0.1:3000
HTTP/1.1 200 OK
Cache-Control: no-store
Content-Type: text/html; charset=UTF-8
X-Content-Type-Options: nosniff
X-Frame-Options: deny
X-Xss-Protection: 1; mode=block
Date: Sun, 11 Jan 2026 16:05:15 GMT

> sudo ss -lptn 'sport = :3000'
State                             Recv-Q                            Send-Q                                                       Local Address:Port                                                       Peer Address:Port                           Process
LISTEN                            0                                 4096                                                                     *:3000                                                                  *:*
> docker ps --format "table {{.Names}}\t{{.Ports}}"
NAMES                             PORTS
app-telemetria-app-a-1            0.0.0.0:8000->8000/tcp
app-telemetria-app-c-1            0.0.0.0:8002->8000/tcp
app-telemetria-app-b-1            0.0.0.0:8001->8000/tcp
app-telemetria-otel-collector-1   0.0.0.0:4317-4318->4317-4318/tcp, 0.0.0.0:8889->8889/tcp, 55678-55679/tcp
app-telemetria-grafana-1          0.0.0.0:3000->3000/tcp
app-telemetria-loki-1             0.0.0.0:3100->3100/tcp
prometheus                        0.0.0.0:9090->9090/tcp
jaeger                            4317-4318/tcp, 0.0.0.0:14250->14250/tcp, 0.0.0.0:14268->14268/tcp, 9411/tcp, 0.0.0.0:6831-6832->6831-6832/udp, 0.0.0.0:16686->16686/tcp
> docker ps -a --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}"
NAMES                             IMAGE                                          PORTS                                                                                                                                     STATUS
app-telemetria-app-a-1            fabriciosveronez/app-telemetria:v1             0.0.0.0:8000->8000/tcp                                                                                                                    Up 2 minutes
app-telemetria-app-c-1            app-telemetria-app-c                           0.0.0.0:8002->8000/tcp                                                                                                                    Up 2 minutes
app-telemetria-app-b-1            app-telemetria-app-b                           0.0.0.0:8001->8000/tcp                                                                                                                    Up 2 minutes
app-telemetria-otel-collector-1   otel/opentelemetry-collector-contrib:0.128.0   0.0.0.0:4317-4318->4317-4318/tcp, 0.0.0.0:8889->8889/tcp, 55678-55679/tcp                                                                 Up 2 minutes
app-telemetria-grafana-1          grafana/grafana:latest                         0.0.0.0:3000->3000/tcp                                                                                                                    Up 3 minutes
app-telemetria-loki-1             grafana/loki:latest                            0.0.0.0:3100->3100/tcp                                                                                                                    Up 3 minutes
prometheus                        prom/prometheus:latest                         0.0.0.0:9090->9090/tcp                                                                                                                    Up 3 minutes
jaeger                            jaegertracing/all-in-one:latest                4317-4318/tcp, 0.0.0.0:14250->14250/tcp, 0.0.0.0:14268->14268/tcp, 9411/tcp, 0.0.0.0:6831-6832->6831-6832/udp, 0.0.0.0:16686->16686/tcp   Up 3 minutes





- Via Explore:
http://localhost:3000/explore

abriu o Grafana