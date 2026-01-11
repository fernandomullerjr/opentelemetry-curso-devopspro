
# 🧭 Guia Prático — Minikube (Instalação e Uso)

## 🎯 Objetivo

* Subir um cluster Kubernetes local
* Trabalhar com `kubectl`
* Usar **Service type: LoadBalancer**
* Ter um ambiente confiável para labs e estudos

---

## 1️⃣ Pré-requisitos

### Obrigatórios

* **Docker** (recomendado como driver)
* **kubectl**
* **Minikube**

Verifique:

```bash
docker --version
kubectl version --client
```

---

## 2️⃣ Instalação do Minikube

### Linux (x86_64)

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

### macOS (Homebrew)

```bash
brew install minikube
```

### Windows (Chocolatey)

```powershell
choco install minikube
```

Verifique:

```bash
minikube version
```

---

## 3️⃣ Subindo o Cluster

### Forma recomendada (Docker driver)

```bash
minikube start --driver=docker
```

📌 **Boas práticas**

* Docker driver é mais leve que VM
* Evita conflitos com VirtualBox/Hyper-V

Verifique:

```bash
kubectl get nodes
```

---

## 4️⃣ Configurações Úteis (Opcional)

### Definir recursos

```bash
minikube start --cpus=4 --memory=8192 --driver=docker
```

### Habilitar addons comuns

```bash
minikube addons enable metrics-server
minikube addons enable ingress
```

Listar addons:

```bash
minikube addons list
```

---

## 5️⃣ Usando o LoadBalancer (PONTO MAIS IMPORTANTE 🔥)

### 🔹 Conceito

Em cloud, o `LoadBalancer` cria um IP externo.
No Minikube, isso é simulado via **tunnel**.

---

### 🔹 Criar um Service LoadBalancer

Exemplo:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: minha-app
spec:
  type: LoadBalancer
  selector:
    app: minha-app
  ports:
    - port: 80
      targetPort: 8080
```

Aplicar:

```bash
kubectl apply -f service.yaml
```

---

### 🔹 Ativar o tunnel

```bash
minikube tunnel
```

⚠️ Normalmente precisa de **sudo**.

Em outro terminal:

```bash
kubectl get svc
```

Você verá:

```text
NAME        TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)
minha-app   LoadBalancer   10.x.x.x        127.0.0.1      80:xxxxx/TCP
```

🎉 Agora o LoadBalancer funciona.

📌 **Importante**

* O `minikube tunnel` precisa ficar rodando
* Ele cria rotas de rede temporárias

---

## 6️⃣ Acessando Serviços

### Via LoadBalancer

```bash
curl http://127.0.0.1
```

### Via NodePort (alternativa)

```bash
minikube service minha-app --url
```

---

## 7️⃣ Dashboard (opcional, mas útil para labs)

```bash
minikube dashboard
```

Abre o painel do Kubernetes no browser.

---

## 8️⃣ Comandos Essenciais do Dia a Dia

```bash
minikube status
minikube stop
minikube start
minikube delete
```

Logs do cluster:

```bash
minikube logs
```

IP do cluster:

```bash
minikube ip
```

---

## 9️⃣ Problemas Comuns e Dicas

### 🔸 LoadBalancer não ganha EXTERNAL-IP

✔️ Verifique se o `minikube tunnel` está rodando

---

### 🔸 Porta não responde

```bash
kubectl describe svc minha-app
kubectl get endpoints minha-app
```

---

### 🔸 Conflito de portas

Evite usar portas já ocupadas no host (3000, 8080, 80).

---

## 🔟 Boas Práticas para Lab Local

✅ Use Docker driver
✅ Use `minikube tunnel` para LB
✅ Prefira `LoadBalancer` para simular cloud
✅ Limpe clusters antigos (`minikube delete`)
❌ Não use Minikube em produção

---

## 🧠 Resumo Mental

* **Minikube** = Kubernetes local completo
* **LoadBalancer** = `minikube tunnel`
* **Ingress** funciona bem junto
* Ideal para **labs, cursos e POCs**



- COMANDOS ÚTEIS NO MINIKUBE:

~~~~BASH
## Abrir Dashboard do k8s
minikube dashboard

## Ativar o tunnel
minikube tunnel

## Fornecer url e porta para acessar o serviço, OBS: Gera porta aleatória acessível
minikube service app-a --url
minikube service grafana --url
~~~~