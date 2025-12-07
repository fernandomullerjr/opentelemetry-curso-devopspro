rascunho.md



https://plataforma.devopspro.com.br/courses/af00ffe1-1d2f-4dce-9a19-780e59273907/2/1/42/conteudos?capitulo=42&conteudo=1232



**Não**, você não precisa rodar comandos `pip` manualmente na pasta.

Analisando o compose.yaml, vejo que:

## **🐳 As Apps Usam Docker Build**

As aplicações (`app-a`, `app-b`, `app-c`) fazem **build** da imagem Docker:

```yaml
build: 
  context: ./src
  dockerfile: ./Dockerfile
```

Isso significa que as dependências Python são instaladas **dentro da imagem Docker** durante o build.

## **📦 Dependências já Incluídas**

O `Dockerfile` (em `./src/Dockerfile`) deve conter algo como:

```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt
```

## **🚀 Como Executar**

Simplesmente rode:

```bash
docker-compose up --build
```

O `--build` garante que as imagens sejam construídas com todas as dependências OpenTelemetry já instaladas.

## **🔍 Verificação**

Se quiser ver quais dependências estão sendo instaladas, verifique:
- `./src/requirements.txt` 
- `./src/Dockerfile`

**Resumo:** O Docker cuida de tudo. As bibliotecas OpenTelemetry já estão configuradas no container.



- Avaliar sobre venv do Python vs Build com Docker, se vai ser necessário algum pip install.
revisar Dockerfile
testar ambiente
```bash
docker-compose up --build
```

> docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
> docker-compose up --build
[+] Running 34/49
 ⠧ otel-collector [⣿⠀⣿] 10.49MB / 89.61MB Pulling                                                                  8.8s
 ⠧ prometheus [⣿⠀⣿⣿⠀⣿⣿⣿⣿⣿] 7.624MB / 134.4MB Pulling                                                               8.8s
 ⠧ loki [⣿⣿⣿⣿⣿⣿⠀⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿] 4.913MB / 41.52MB Pulling                                                              8.8s
 ⠧ grafana [⣿⣿⠀⣿⣤⣄⣿⣿⠀] 21.08MB / 206.3MB Pulling                                                                   8.8s
 ⠧ jaeger [⣿⣿⡀⣀⣿] 7.467MB / 37.39MB Pulling                                                                        8.8s


> curl http://localhost:8000/
{"message":"Esse é o serviço app-a"}%                                                                                   > date
Sun Nov 30 14:22:07 -03 2025




## Pendente

- Avaliar sobre venv do Python vs Build com Docker, se vai ser necessário algum pip install.
revisar Dockerfile
testar ambiente
```bash
docker-compose up --build
```

- Revisar porta 3000
tá jogando pro Open WebUI
http://localhost:3000/

- Finalizar aula "O que são métricas?"
em 06:56min

- Finalizar aula "Configuração do OpenTelemetry"
em 04:53min




## Dia 06/12/2025

- Parei o Open WebUI
docker stop open-webui

- Subindo stack inteira:
docker compose up --build

- Containers subiram:

> docker ps
CONTAINER ID   IMAGE                                          COMMAND                  CREATED          STATUS          PORTS                                                                                                                                     NAMES
ce7c4d5aa326   app-telemetria-app-b                           "uvicorn app:app --h…"   18 seconds ago   Up 16 seconds   0.0.0.0:8001->8000/tcp                                                                                                                    app-telemetria-app-b-1
14e6158faa15   fabriciosveronez/app-telemetria:v1             "uvicorn app:app --h…"   18 seconds ago   Up 16 seconds   0.0.0.0:8000->8000/tcp                                                                                                                    app-telemetria-app-a-1
17a174c0ccb4   app-telemetria-app-c                           "uvicorn app:app --h…"   18 seconds ago   Up 16 seconds   0.0.0.0:8002->8000/tcp                                                                                                                    app-telemetria-app-c-1
ecd341c80ed7   otel/opentelemetry-collector-contrib:0.128.0   "/otelcol-contrib --…"   6 days ago       Up 16 seconds   0.0.0.0:4317-4318->4317-4318/tcp, 0.0.0.0:8889->8889/tcp, 55678-55679/tcp                                                                 app-telemetria-otel-collector-1
e270f4748d92   grafana/grafana:latest                         "sh -euc 'mkdir -p /…"   6 days ago       Up 17 seconds   0.0.0.0:3000->3000/tcp                                                                                                                    app-telemetria-grafana-1
3778824a249f   grafana/loki:latest                            "/usr/bin/loki -conf…"   6 days ago       Up 17 seconds   0.0.0.0:3100->3100/tcp                                                                                                                    app-telemetria-loki-1
96fdcafdb849   prom/prometheus:latest                         "/bin/prometheus --c…"   6 days ago       Up 17 seconds   0.0.0.0:9090->9090/tcp                                                                                                                    prometheus
887999dcb557   jaegertracing/all-in-one:latest                "/go/bin/all-in-one-…"   6 days ago       Up 17 seconds   4317-4318/tcp, 0.0.0.0:14250->14250/tcp, 0.0.0.0:14268->14268/tcp, 9411/tcp, 0.0.0.0:6831-6832->6831-6832/udp, 0.0.0.0:16686->16686/tcp   jaeger

 ~                                                                                                                                                                                                                                                                        ok  21:48:25



abriu grafana:
http://localhost:3000/
http://localhost:3000/?orgId=1&from=now-6h&to=now&timezone=browser



- Finalizar aula "O que são métricas?"
em 06:56min

Counter
Gauge
Histogram
Summary



- Avaliar sobre venv do Python vs Build com Docker, se vai ser necessário algum pip install.
revisar Dockerfile
testar ambiente
```bash
docker-compose up --build
```