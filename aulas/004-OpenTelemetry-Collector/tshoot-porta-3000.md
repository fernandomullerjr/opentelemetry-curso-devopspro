


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












segue querendo abrir o open-webui mesmo removendo todas sujeiras dele


> kubectl get pods -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS        AGE
default       app-a-74f7747c78-7h8nv             1/1     Running   0               104s
default       app-b-6bd9d84f56-gzvhp             1/1     Running   0               104s
default       app-c-776f947589-w2dh6             1/1     Running   0               104s
default       grafana-bfb9674f7-hjpkx            1/1     Running   0               104s
default       jaeger-766cc89bdf-wbgjk            1/1     Running   0               104s
default       loki-c8d9f66d4-wwhwr               1/1     Running   0               104s
default       prometheus-68fd4f969-jswpg         1/1     Running   0               103s
kube-system   coredns-66bc5c9577-rmfg7           1/1     Running   0               6m13s
kube-system   etcd-minikube                      1/1     Running   0               6m18s
kube-system   kube-apiserver-minikube            1/1     Running   0               6m18s
kube-system   kube-controller-manager-minikube   1/1     Running   0               6m18s
kube-system   kube-proxy-ljs8m                   1/1     Running   0               6m13s
kube-system   kube-scheduler-minikube            1/1     Running   0               6m18s
kube-system   storage-provisioner                1/1     Running   1 (5m51s ago)   6m17s
> kubectl get svc
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                      AGE
app-a        LoadBalancer   10.107.151.221   127.0.0.1     80:30001/TCP                                                                 110s
app-b        ClusterIP      10.107.150.105   <none>        8000/TCP                                                                     110s
app-c        ClusterIP      10.99.50.206     <none>        8000/TCP                                                                     110s
grafana      LoadBalancer   10.110.95.217    127.0.0.1     3000:30000/TCP                                                               110s
jaeger       ClusterIP      10.107.72.102    <none>        16686/TCP,14250/TCP,14268/TCP,6831/UDP,6832/UDP,9411/TCP,4317/TCP,4318/TCP   110s
kubernetes   ClusterIP      10.96.0.1        <none>        443/TCP                                                                      6m25s
loki         ClusterIP      10.107.169.227   <none>        3100/TCP                                                                     110s
prometheus   NodePort       10.109.37.53     <none>        9090:30002/TCP                                                               110s
> ps -ef | grep 3000
root        1323     675  0 13:39 ?        00:00:00 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 3000 -container-ip 172.18.0.2 -container-port 8080
> kill -9 1323
kill: kill 1323 failed: operation not permitted
> sudo kill -9 1323
>


docker fica criando este docker-proxy
mesmo fazendo kill, fica tentando abrir ele.
como eliminar ele totalmente?





Investigando, não tem nada sobre esta rede


> docker ps -aq | while read id; do
  name=$(docker inspect -f '{{.Name}}' "$id" 2>/dev/null)
  img=$(docker inspect -f '{{.Config.Image}}' "$id" 2>/dev/null)
  ips=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}} {{end}}' "$id" 2>/dev/null)
  printf "%s\t%s\t%s\n" "$name" "$ips" "$img"
done | sort

/app-telemetria-app-a-1         fabriciosveronez/app-telemetria:v1
/app-telemetria-app-b-1         app-telemetria-app-b
/app-telemetria-app-c-1         app-telemetria-app-c
/app-telemetria-grafana-1               grafana/grafana:latest
/app-telemetria-loki-1          grafana/loki:latest
/app-telemetria-otel-collector-1                otel/opentelemetry-collector-contrib:0.128.0
/jaeger         jaegertracing/all-in-one:latest
/minikube       192.168.49.2    gcr.io/k8s-minikube/kicbase:v0.0.48@sha256:7171c97a51623558720f8e5878e4f4637da093e2f2ed589997bedc6c1549b2b1
/prometheus             prom/prometheus:latest
> docker network ls -q | while read n; do
  docker network inspect "$n" --format '{{.Name}} {{range .IPAM.Config}}{{.Subnet}}{{end}}'
done | sort

app-telemetria_app-network 172.18.0.0/16
bridge 172.17.0.0/16
host
minikube 192.168.49.0/24
none
> docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}"
NAMES      IMAGE                                 PORTS                                                                                                                                  STATUS
minikube   gcr.io/k8s-minikube/kicbase:v0.0.48   127.0.0.1:51253->22/tcp, 127.0.0.1:51254->2376/tcp, 127.0.0.1:51256->5000/tcp, 127.0.0.1:51257->8443/tcp, 127.0.0.1:51255->32443/tcp   Up 10 minutes
> systemctl list-units --type=service | grep -i docker
  cri-docker.service                                     loaded active running CRI Interface for Docker Application Container Engine
  docker.service                                         loaded active running Docker Application Container Engine
> systemctl list-units --type=service | grep -Ei "webui|open|ollama|ai|ui"
  containerd.service                                     loaded active running containerd container runtime
  cri-docker.service                                     loaded active running CRI Interface for Docker Application Container Engine
  docker.service                                         loaded active running Docker Application Container Engine
  ollama.service                                         loaded active running Ollama Service
  plymouth-quit-wait.service                             loaded active exited  Hold until boot process finishes up
  plymouth-quit.service                                  loaded active exited  Terminate Plymouth Boot Screen
  snap.ubuntu-desktop-installer.subiquity-server.service loaded active running Service for snap application ubuntu-desktop-installer.subiquity-server
  snapd.seeded.service                                   loaded active exited  Wait until snapd is fully seeded
  systemd-networkd-wait-online.service                   loaded active exited  Wait for Network to be Configured
  user@0.service                                         loaded active running User Manager for UID 0
  user@1000.service                                      loaded active running User Manager for UID 1000
> sudo lsof -nP -iTCP:3000 -sTCP:LISTEN
> docker ps -aq | while read id; do
  ips=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}} {{end}}' "$id" 2>/dev/null)
  if echo "$ips" | grep -q "172.18.0.2"; then
    docker inspect -f 'NAME={{.Name}} ID={{.Id}} IMAGE={{.Config.Image}} RESTART={{.HostConfig.RestartPolicy.Name}} PORTS={{json .HostConfig.PortBindings}}' "$id"
  fi
done










> PID=$(sudo lsof -nP -iTCP:3000 -sTCP:LISTEN | awk 'NR==2{print $2}')
sudo tr '\0' ' ' < /proc/$PID/cmdline ; echo

initrd=\initrd.img WSL_ROOT_INIT=1 panic=-1 nr_cpus=16 hv_utils.timesync_implicit=1 console=hvc0 debug pty.legacy_count=0 WSL_ENABLE_CRASH_DUMP=1

> docker ps -aq | while read id; do
  ips=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}} {{end}}' "$id" 2>/dev/null)
  if echo "$ips" | grep -q "<IP_DO_CONTAINER>"; then
    docker inspect -f 'NAME={{.Name}} ID={{.Id}} IMAGE={{.Config.Image}} RESTART={{.HostConfig.RestartPolicy.Name}} PORTS={{json .HostConfig.PortBindings}}' "$id"
  fi
done

> sudo lsof -nP -iTCP:3000 -sTCP:LISTEN | awk 'NR==2{print $2}'
> sudo iptables -t nat -S | grep -E '3000|REDIRECT|DNAT' || true
sudo iptables -S | grep -E '3000' || true

-A DOCKER ! -i docker0 -p tcp -m tcp --dport 3000 -j DNAT --to-destination 172.18.0.2:8080
> ip -4 addr show | grep -E 'inet '
sudo lsof -nP -iTCP:3000 -sTCP:LISTEN
curl -sv http://192.168.1.30:3000/ 2>&1 | head -n 25

    inet 127.0.0.1/8 scope host lo
    inet 192.168.1.30/24 brd 192.168.1.255 scope global eth0
    inet 192.168.0.107/24 brd 192.168.0.255 scope global eth0
    inet 172.18.0.1/16 brd 172.18.255.255 scope global docker0
*   Trying 192.168.1.30:3000...
* Connected to 192.168.1.30 (192.168.1.30) port 3000 (#0)
> GET / HTTP/1.1
> Host: 192.168.1.30:3000
> User-Agent: curl/7.81.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< date: Sun, 11 Jan 2026 17:01:33 GMT
< server: uvicorn
< content-type: text/html; charset=utf-8
< accept-ranges: bytes
< content-length: 6952
< last-modified: Thu, 28 Aug 2025 10:43:49 GMT
< etag: "95ade68d267ede64c01e1534ce7ebc72"
< vary: Accept-Encoding
< x-process-time: 0
<
{ [6952 bytes data]
<!doctype html>
<html lang="en">
        <head>
                <meta charset="utf-8" />
                <link rel="icon" type="image/png" href="/static/favicon.png" />



5) Se o host NÃO tem nada ouvindo, então há regra de NAT/forwarding

Isso acontece quando iptables redireciona 3000 para outro lugar.

Cheque regras que mexem em 3000:

sudo iptables -t nat -S | grep -E '3000|REDIRECT|DNAT' || true
sudo iptables -S | grep -E '3000' || true



> sudo lsof -nP -iTCP:3000 -sTCP:LISTEN | awk 'NR==2{print $2}'
> sudo iptables -t nat -S | grep -E '3000|REDIRECT|DNAT' || true
sudo iptables -S | grep -E '3000' || true

-A DOCKER ! -i docker0 -p tcp -m tcp --dport 3000 -j DNAT --to-destination 172.18.0.2:8080






Achou o culpado: **não é um processo escutando na porta 3000**. É uma **regra NAT do Docker no iptables** que está redirecionando tráfego para **`172.18.0.2:8080`**:

```
-A DOCKER ! -i docker0 -p tcp -m tcp --dport 3000 -j DNAT --to-destination 172.18.0.2:8080
```

Isso explica tudo:

* `lsof` vazio (ninguém está “LISTEN” no host)
* mesmo assim `curl 192.168.1.30:3000` responde (iptables está fazendo DNAT)
* resposta `server: uvicorn` (Open WebUI)
* container “já foi removido” → ficou **regra órfã**/sujeira no iptables

Agora vamos remover **de forma definitiva**.

---

## ✅ 1) Remover a regra órfã do iptables (imediato)

Execute exatamente:

```bash
sudo iptables -t nat -D DOCKER ! -i docker0 -p tcp -m tcp --dport 3000 -j DNAT --to-destination 172.18.0.2:8080
```

Confirme que sumiu:

```bash
sudo iptables -t nat -S DOCKER | grep 3000 || echo "OK: sem regra 3000"
```

Teste:

```bash
curl -I http://192.168.1.30:3000
```

Agora deve dar **connection refused** (ou timeout), a menos que você tenha Grafana/Minikube expondo 3000 de alguma forma.

---

## ✅ 2) Evitar que a regra volte (reiniciando Docker limpa e reescreve as regras corretas)

Depois de remover, reinicie o Docker para ele reprogramar as chains corretamente:

```bash
sudo systemctl restart docker
```

E verifique novamente:

```bash
sudo iptables -t nat -S DOCKER | grep 3000 || echo "OK: sem regra 3000"
```

---

## ✅ 3) Se você quer “zerar tudo do Docker” exceto Minikube/K8s

Como você está no **WSL** (apareceu `WSL_ROOT_INIT=1`), e Minikube (driver docker) cria/usa regras também, o “limpar geral” tem que ser feito com cuidado.

O caminho seguro é:

### 3.1 Derrubar containers/compose antigos

```bash
docker ps -a
```

Remova o que não for necessário.

### 3.2 Limpar redes não usadas (opcional)

```bash
docker network prune -f
```

### 3.3 Reiniciar Docker (recria as chains)

```bash
sudo systemctl restart docker
```

---

## ✅ 4) Como garantir que só o Minikube/K8s exponha portas

Se a sua intenção é: **nada do Docker expor na LAN**, só o que vier do Kubernetes:

* Não use `docker run -p ...` fora do minikube
* Se estiver usando `docker compose` para observability, ele vai expor portas direto no host.
* No k8s, prefira:

  * `minikube service <svc> --url` (acesso local)
  * `NodePort`/`LoadBalancer` com `minikube tunnel` (controlado)

---

## ⚠️ Observação importante

Você tem **2 IPs na mesma interface** (`192.168.1.30` e `192.168.0.107`) — isso pode criar confusão de roteamento, mas não é a causa aqui. A causa foi **DNAT explícito na chain DOCKER**.

---

## Se a porta 3000 ainda abrir depois disso

Aí pode existir regra semelhante em **DOCKER-USER** ou no **PREROUTING**:

```bash
sudo iptables -t nat -S | grep 3000
sudo iptables -S DOCKER-USER | grep 3000 || true
```

Mas pelo que você mostrou, a regra principal já está identificada e é suficiente remover.

---

Se você rodar os comandos do passo **1** e colar o output do `curl -I http://192.168.1.30:3000` depois, eu confirmo se ficou limpo e te digo como voltar o Grafana na 3000 (via k8s ou docker) sem risco de “ressuscitar” o Open WebUI.
