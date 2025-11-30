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





## Pendente

- Avaliar sobre venv do Python vs Build com Docker, se vai ser necessário algum pip install.
revisar Dockerfile
testar ambiente

- Finalizar aula "O que são métricas?"
em 06:56min

- Finalizar aula "Configuração do OpenTelemetry"
em 04:53min