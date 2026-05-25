# Setup Local: Dify + Ollama (Agente de Notícias)
 
Este repositório documenta a investigação, instalação e validação da plataforma [Dify](https://github.com/langgenius/dify) rodando localmente com modelos via Ollama, focado na criação de um agente curador de notícias de tecnologia.
 
## 🖥️ Ambiente de Execução Local
 
| Componente | Especificação |
|---|---|
| **GPU** | AMD Radeon RX 6600 — 8GB |
| **CPU** | AMD Ryzen 5 5600 |
| **RAM** | 32GB |
| **Monitoramento de VRAM** | `watch -n 1 rocm-smi` |
 
---
 
## 🚀 1. Instalar e Configurar Ollama
 
Instale o Ollama via terminal:
 
```bash
curl -fsSL https://ollama.com/install.sh | sh
```
 
Valide se o serviço está ativo rodando em background:
 
```bash
curl http://localhost:11434/api/tags
```
 
Baixe a stack de modelos que será utilizada no benchmark:
 
```bash
ollama pull qwen2.5-coder:7b
ollama pull llama3.1:8b
ollama pull mistral:7b
ollama pull deepseek-r1:8b
```
 
---
 
## 🐳 2. Instalar e Subir o Dify via Docker Compose
 
Clone o repositório oficial do Dify:
 
```bash
git clone https://github.com/langgenius/dify.git
```
 
Acesse o diretório de deploy, prepare as variáveis de ambiente e suba os containers:
 
```bash
cd dify/docker
cp .env.example .env
docker compose up -d
```
 
> **Nota:** Foi utilizado o `docker-compose.yaml` padrão do repositório (`langgenius/dify`). Os serviços de banco (PostgreSQL), cache (Redis) e vetores (Weaviate) subiram com as configurações default carregadas via `.env`.
 
Após finalizar, a interface de setup inicial ficará disponível em:
**http://localhost/install**
 
---
 
## 🔌 3. Configuração do Ollama Provider no Dify
 
Para conectar o Dify aos modelos locais, foi instalado o plugin oficial do Ollama direto na interface.
 
**Caminho:** `Settings → Model Provider → Install Ollama Plugin`
 
**Parâmetros utilizados na configuração:**
 
| Parâmetro | Valor |
|---|---|
| **Base URL** | `http://172.17.0.1:11434` |
| **Porta** | `11434` |
 
> **Por que `172.17.0.1`?** Foi utilizado o IP da bridge do Docker no Linux como garantia de conexão, caso o `host.docker.internal` não resolva na máquina host.
 
## 🧪 4. Testes Realizados

Abaixo estão os cenários de testes validados na máquina local. Cada link direciona para a documentação detalhada com os prints, logs e consumo de hardware.

* [Teste 01: Chatbot Local Inicial (Qwen 2.5)](Tarefas/TESTEINICIAL.md) — Validação básica de resposta offline e comprovação de consumo da GPU (RX 6600).