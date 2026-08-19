# Guia Completo: Ollama (ROCm), Open WebUI e Hermes Agent no Bluefin Linux

Este guia prático foi criado especificamente para configurar um ecossistema completo de Inteligência Artificial local no seu **Mini PC GEEKOM A7 MAX** rodando o sistema **Bluefin Linux**. 

Aqui, nós vamos instalar de forma automatizada via **Docker**:
1. **Ollama (com ROCm)**: O motor de processamento local, configurado para usar toda a potência da sua placa de vídeo integrada **Radeon 780M** (arquitetura `gfx1103` emulada como `gfx1100`).
2. **Open WebUI**: A interface visual estilo ChatGPT para você conversar com seus modelos no navegador.
3. **Hermes Agent (Nous Research)**: O poderoso agente autônomo que aprende novas habilidades, gerencia tarefas e pode ser conectado ao Telegram, Discord, Slack ou usado diretamente no terminal.

---

## 🛠️ Requisitos Iniciais

Como você possui **64 GB de RAM**, a sua placa integrada Radeon 780M pode pegar emprestado dinamicamente até **32 GB de memória gráfica (VRAM)**. Para garantir que o sistema não limite essa alocação, abra o terminal do seu Bluefin e execute o comando abaixo (isso configurará o driver AMD de forma perfeita para IA):

```bash
sudo systemctl enable docker
```

---

## 🚀 Passo 1: Criando a Pasta do Projeto

Abra o seu terminal no Bluefin Linux e crie uma pasta organizada para hospedar toda a nossa estrutura do Docker:

```bash
mkdir -p ~/ia-local && cd ~/ia-local
```

---

## 📦 Passo 2: Criando o Arquivo de Configuração Único (`docker-compose.yml`)

O Docker Compose permite subir múltiplos serviços com apenas uma "receita". Vamos criar esse arquivo dentro da pasta que acabamos de criar.

1. Digite no terminal para abrir o editor:
   ```bash
   nano docker-compose.yml
   ```

2. Copie todo o código abaixo, cole dentro do editor de texto e salve pressionando **Ctrl + O**, depois **Enter** para confirmar, e saia com **Ctrl + X**:

```yaml
version: '3.8'

services:
  # 1. Motor Ollama com suporte a aceleração AMD ROCm
  ollama:
    image: ollama/ollama:rocm
    container_name: ollama
    restart: unless-stopped
    ports:
      - "11434:11434"
    volumes:
      - ollama_data:/root/.ollama
    devices:
      - "/dev/kfd:/dev/kfd"
      - "/dev/dri:/dev/dri"
    environment:
      - HSA_OVERRIDE_GFX_VERSION=11.0.0 # Engana o ROCm para ativar a Radeon 780M
    group_add:
      - video

  # 2. Interface Visual estilo ChatGPT (Open WebUI)
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: unless-stopped
    ports:
      - "3000:8080"
    volumes:
      - open_webui_data:/app/backend/data
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
    depends_on:
      - ollama

  # 3. Hermes Agent (Nous Research) - Agente Autônomo
  hermes-agent:
    image: nousresearch/hermes-agent:latest
    container_name: hermes-agent
    restart: unless-stopped
    ports:
      - "8642:8642"
    volumes:
      - ~/.hermes:/opt/data
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
    depends_on:
      - ollama

volumes:
  ollama_data:
  open_webui_data:
```

---

## 🧙‍♂️ Passo 3: Rodar o Assistente de Configuração do Hermes Agent

O **Hermes Agent** é um agente de inteligência autônoma. Na primeira vez que você for usá-lo, ele precisa rodar um assistente interativo para você definir as suas preferências (como chaves de API, se deseja conectá-lo ao Telegram, Discord, etc.).

1. Antes de ligar todo o sistema, crie a pasta onde o Hermes guardará suas memórias e chaves na sua máquina host:
   ```bash
   mkdir -p ~/.hermes
   ```

2. Execute o assistente de configuração oficial do Hermes rodando este comando no terminal:
   ```bash
   docker run -it --rm -v ~/.hermes:/opt/data nousresearch/hermes-agent setup
   ```
   * O assistente fará perguntas simples na tela. Ele perguntará qual modelo você quer usar (você pode selecionar o Ollama local ou provedores externos como OpenRouter, Anthropic, etc.), suas chaves e se você deseja ativar canais de mensagem como o Telegram.
   * Assim que você concluir o assistente, ele salvará de forma segura as suas configurações na sua pasta local `~/.hermes/.env` e fechará o assistente automaticamente.

---

## ⚡ Passo 4: Subir Todo o Sistema com 1 Comando

Com as receitas prontas e as configurações do Hermes salvas, basta rodar este comando para subir o Ollama, o Open WebUI e o Hermes Agent de uma única vez em segundo plano:

```bash
docker compose up -d
```

O Docker fará o download de todas as imagens oficiais prontas direto do Docker Hub. Isso pode levar alguns minutos dependendo da sua velocidade de internet. Assim que terminar, você verá mensagens de "Started" na tela.

---

## 🎮 Como Usar o seu Novo Setup de IA

### 1. Interface de Chat (Open WebUI)
Abra o seu navegador de internet e acesse:
👉 **`http://localhost:3000`**

Crie o seu primeiro usuário local (seus dados ficam salvos de forma 100% privada no seu próprio mini PC). Na aba de configurações, você poderá baixar e gerenciar qualquer modelo do Ollama de forma visual!

### 2. Conversar ou Controlar o Hermes Agent
O Hermes Agent estará rodando silenciosamente na porta `8642` e utilizando as configurações que você definiu no assistente (como o seu bot do Telegram ou Discord). 

Se você quiser abrir o terminal interativo do Hermes para dar ordens diretas para ele rodar na sua máquina, basta executar o comando abaixo:
```bash
docker exec -it hermes-agent hermes
```

---

## 🔄 Como Atualizar Tudo no Futuro?

Sempre que a AMD, a Nous Research ou o time do Open WebUI lançarem atualizações, você pode atualizar todos os seus aplicativos do Docker de uma vez com estes três comandos simples dentro da pasta `~/ia-local`:

```bash
cd ~/ia-local
docker compose pull
docker compose up -d
```

Suas conversas, preferências e memórias de agente nunca serão perdidas ao atualizar, pois estão protegidas em volumes persistentes do seu computador!
