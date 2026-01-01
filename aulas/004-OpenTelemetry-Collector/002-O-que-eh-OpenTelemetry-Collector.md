
## O que é o OpenTelemetry Collector


Otel Collector

OTLP

Disponibilidade

Replicas do OTEL Collector
Load Balancer
OTEL Gateway

Alternativa:
sidecar(tem latencia baixa como vantagem, complexidade e uso maior de recurso como desvantagem. recomendo apenas quando a aplicação precisa de rapidez no envio de dados)


Receiver
define como vai receber os dados das aplicações
precisa escolher HTTP ou gRPC

Processor
faz a manipulação dos dados.

Exporter
define para qual ferramenta irão os dados

Pipeline
define o fluxo de todos estes elementos



---

# 📘 Guia de Aula — O que é o OpenTelemetry Collector

## 🎯 Objetivo da aula

Explicar **o papel do OpenTelemetry Collector**, como ele se encaixa na arquitetura de observabilidade, seus componentes internos e **boas práticas de uso em produção**.

---

## 1️⃣ O que é o OpenTelemetry Collector (OTel Collector)

O **OpenTelemetry Collector** é um **serviço intermediário** responsável por:

* Receber dados de observabilidade (traces, métricas, logs)
* Processar esses dados (filtrar, enriquecer, agrupar)
* Exportar para backends (Tempo, Jaeger, Prometheus, Datadog, etc.)

👉 Ele **desacopla a aplicação do backend**, evitando que cada app precise conhecer detalhes de exportação.

**Mensagem-chave para a aula:**

> Aplicações enviam dados → Collector decide o que fazer com eles.

---

## 2️⃣ OTLP (OpenTelemetry Protocol)

### O que é

O **OTLP** é o **protocolo padrão** do OpenTelemetry para envio de dados.

* Suporta **traces, metrics e logs**
* Pode usar:

  * **gRPC** (binário, mais eficiente)
  * **HTTP/JSON** (mais simples e compatível)

### Dica didática

* **gRPC** → melhor desempenho e menor overhead
* **HTTP** → mais fácil para debug e compatibilidade

📌 **Boa prática**:

> Sempre que possível, use **OTLP/gRPC**.

---

## 3️⃣ Disponibilidade e Alta Confiabilidade

O Collector é um **componente crítico**:

* Se ele cair, os dados podem ser perdidos
* Não deve ser ponto único de falha

### Estratégias

* Múltiplas réplicas
* Load Balancer
* Uso de Gateway

---

## 4️⃣ Réplicas do OTEL Collector

Rodar múltiplas instâncias do Collector permite:

* Alta disponibilidade
* Escalabilidade horizontal
* Menor impacto de falhas

📌 **Boa prática**:

* Stateless sempre que possível
* Usar `batch processor` para reduzir carga no backend

---

## 5️⃣ Load Balancer

Usado quando:

* Várias aplicações enviam dados para o Collector
* Existem múltiplas réplicas

Funções:

* Distribuir carga
* Evitar sobrecarga em um único Collector

💡 Exemplo:

* Service do Kubernetes
* ALB / NLB
* Envoy / Nginx

---

## 6️⃣ OTEL Gateway

### O que é

O **OTEL Gateway** é um Collector rodando como **camada central**.

Arquitetura comum:

```
Aplicações → Collector Agent → OTEL Gateway → Backend
```

### Vantagens

* Centraliza regras de processamento
* Facilita governança
* Reduz impacto nas aplicações

📌 **Boa prática**:

> Use Gateway quando há muitos serviços ou múltiplos times.

---

## 7️⃣ Alternativa: Sidecar

### O que é

O Collector roda **junto da aplicação**, no mesmo pod/host.

### Vantagens

* Latência muito baixa
* Menos dependência de rede

### Desvantagens

* Maior consumo de recursos
* Mais complexidade operacional
* Difícil escalar e manter

📌 **Recomendação clara para a aula**:

> **Use sidecar apenas quando a aplicação exige envio extremamente rápido de dados.**
> Para a maioria dos casos, **agent + gateway é melhor**.

---

## 8️⃣ Componentes Internos do Collector

### 🔹 Receiver

Define **como os dados chegam** ao Collector.

Exemplos:

* `otlp` (HTTP ou gRPC)
* `prometheus`
* `jaeger`

📌 Ponto de atenção:

* Escolher **HTTP ou gRPC**
* Verificar portas e segurança

---

### 🔹 Processor

Responsável por **manipular os dados**.

Exemplos comuns:

* `batch` → agrupa dados (essencial)
* `attributes` → adiciona/remove campos
* `filter` → descarta dados irrelevantes
* `memory_limiter` → evita crash por OOM

📌 **Boa prática obrigatória**:

> Sempre usar `batch` + `memory_limiter`.

---

### 🔹 Exporter

Define **para onde os dados vão**.

Exemplos:

* `otlp` (para gateway ou outro collector)
* `jaeger`
* `prometheus`
* `logging` (debug)

📌 Dica:

* Use `logging exporter` apenas para debug, nunca em produção.

---

## 9️⃣ Pipeline

O **pipeline** conecta tudo:

```
Receiver → Processor → Exporter
```

Exemplo mental:

> “Recebo via OTLP → processo em batch → envio para Tempo”

Cada sinal (traces, metrics, logs) tem seu próprio pipeline.

---

## 🔟 Boas Práticas Gerais (Resumo)

✅ Use **OTLP/gRPC**
✅ Evite Collector único (SPOF)
✅ Prefira **Agent + Gateway**
✅ Sempre use `batch` e `memory_limiter`
✅ Centralize regras no Gateway
❌ Evite sidecar sem necessidade real
❌ Não exponha Collector sem autenticação

---

## 🧠 Frase final para a aula

> **O OpenTelemetry Collector é o cérebro da observabilidade: ele decide o que entra, como é tratado e para onde vai.**


