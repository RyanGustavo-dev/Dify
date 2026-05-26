# 🛠️ Troubleshooting: Dify + Ollama — `host.docker.internal` Connection Refused

Guia de resolução para o erro de conexão entre o Dify (Docker) e o Ollama (host) no **Linux** e **WSL2**.

---

## ❌ Problema

Ao configurar o Ollama como provider no Dify, o seguinte erro aparece ao salvar o modelo:

```
An error occurred during credentials validation:
HTTPConnectionPool(host='host.docker.internal', port=11434):
Max retries exceeded with url: /api/chat
(Caused by NewConnectionError: Failed to establish a new connection: Connection refused)
```

---

## 🔍 Causa Raiz

O erro ocorre por **duas razões combinadas**, comuns no Linux e WSL2:

| Causa | Descrição |
|---|---|
| `host.docker.internal` não resolve | No Linux, esse hostname não é mapeado automaticamente para o host (ao contrário do Windows/Mac) |
| Ollama vinculado a `127.0.0.1` | Por padrão, o Ollama escuta apenas na interface loopback, inacessível de dentro dos containers |

---

## ✅ Solução Completa

### Passo 1 — Verificar o estado atual do Ollama

```bash
ss -tlnp | grep 11434
```

Se aparecer `127.0.0.1:11434`, o Ollama está inacessível pelos containers. Siga para o Passo 2.

---

### Passo 2 — Expor o Ollama em todas as interfaces

Crie o arquivo de override do systemd:

```bash
sudo mkdir -p /etc/systemd/system/ollama.service.d

sudo tee /etc/systemd/system/ollama.service.d/override.conf << 'EOF'
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
EOF
```

Recarregue e reinicie:

```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

Valide:

```bash
ss -tlnp | grep 11434
# Esperado: *:11434   (e não 127.0.0.1:11434)
```

> **WSL2:** Se o `systemctl` não funcionar, use diretamente:
> ```bash
> pkill ollama
> OLLAMA_HOST=0.0.0.0:11434 ollama serve &
> ```

---

### Passo 3 — Mapear `host.docker.internal` no Docker Compose

No Linux, o hostname `host.docker.internal` não é resolvido automaticamente. É necessário adicioná-lo manualmente nos serviços do `docker-compose.yaml`.

Abra o arquivo `dify/docker/docker-compose.yaml` e adicione `extra_hosts` nos serviços `api`, `worker` e `plugin_daemon`:

```yaml
api:
  # ... configurações existentes ...
  extra_hosts:
    - "host.docker.internal:host-gateway"

worker:
  # ... configurações existentes ...
  extra_hosts:
    - "host.docker.internal:host-gateway"

plugin_daemon:
  # ... configurações existentes ...
  extra_hosts:
    - "host.docker.internal:host-gateway"
```

Reinicie os containers:

```bash
docker compose down
docker compose up -d
```

---

### Passo 4 — Validar a conexão de dentro do container

```bash
docker exec -it docker-api-1 curl http://host.docker.internal:11434/api/tags
```

Resposta esperada (JSON com os modelos instalados):

```json
{"models":[{"name":"qwen2.5-coder:7b","..."}]}
```

---

### Passo 5 — Configurar o provider no Dify

Com a conexão funcionando, acesse o Dify e configure:

```
Settings → Model Provider → Ollama → Adicionar Modelo
```

| Campo | Valor |
|---|---|
| Base URL | `http://host.docker.internal:11434` |
| Model Name | `qwen2.5-coder:7b` (ou o modelo desejado) |
| Model Type | `LLM` |

> **Alternativa:** Se preferir não usar `host.docker.internal`, use o IP direto da bridge Docker:
> ```bash
> ip addr show docker0 | grep "inet "
> # Geralmente: 172.17.0.1
> ```
> E configure a Base URL como `http://172.17.0.1:11434`.

---

## 📋 Diagnóstico Rápido

| Sintoma | Causa | Fix |
|---|---|---|
| `Could not resolve host: host.docker.internal` | `extra_hosts` ausente no compose | Passo 3 |
| `Failed to connect to host.docker.internal port 11434` | Ollama em `127.0.0.1` | Passo 2 |
| `model 'X' not found` | Modelo não baixado | `ollama pull <model>` |

---

## 🧪 Ambiente Validado

- Ubuntu 22.04 / WSL2 (Windows 11)
- Docker Engine 27+
- Dify `1.14.2` (docker-compose padrão do repositório)
- Ollama `0.6+`

---

## 📦 Modelos Recomendados para Começar

```bash
ollama pull qwen2.5-coder:7b   # coding / agentes
ollama pull llama3.1:8b        # uso geral
ollama pull mistral:7b         # rápido e leve
ollama pull deepseek-r1:8b     # raciocínio
```
