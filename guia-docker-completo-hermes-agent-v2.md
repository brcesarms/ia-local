# Guia Definitivo: Hermes Agent + Open WebUI alimentados pelo LM Studio (Vulkan)

Este guia prático ensina como configurar uma estrutura de Inteligência Artificial local unindo o **LM Studio** rodando diretamente na sua máquina host (aproveitando o motor Vulkan super veloz para placas AMD) [3, 10] e o **Hermes Agent** + **Open WebUI** rodando em containers **Docker** no seu **GEEKOM A7 MAX** com **Bluefin Linux**.

Ao usar o LM Studio como o "cérebro" (servidor de modelos) [3], nós eliminamos 100% da necessidade de configurar o ROCm ou de lutar contra permissões de drivers de GPU dentro de containers [3]. O LM Studio gerencia a placa integrada **Radeon 780M** de forma nativa e extremamente rápida (alcançando entre **20 a 23 tokens por segundo** em chips integrados AMD RDNA3) [3].

---

## 🏗️ Como a arquitetura funciona?

1. **LM Studio (na Máquina Host)**: Carrega os modelos de IA e expõe uma API compatível com a OpenAI rodando na porta local `1234` [3].
2. **Docker Network Bridge (`host.docker.internal`)**: Uma ponte de rede que permite aos aplicativos dentro do Docker (Hermes e Open WebUI) acessarem de forma segura o servidor de IA rodando diretamente no seu sistema.
3. **Open WebUI (em Docker)**: A interface visual bonita de chat acessível no seu navegador.
4. **Hermes Agent (em Docker)**: O seu agente autônomo oficial que gerencia ferramentas, executa tarefas e gerencia memórias.

---

## ⚡ Passo 1: Configurar o LM Studio (O Cérebro)

Primeiro, precisamos ativar o servidor de IA dentro do seu LM Studio:

1. Abra o **LM Studio** no seu Bluefin Linux.
2. Baixe o modelo desejado (recomenda-se o **Hermes 3 8B GGUF** ou similar de sua preferência).
3. Vá até a aba **Local Server** (representada pelo ícone de duas setas no menu lateral esquerdo do aplicativo).
4. No painel de configurações à direita, garanta que a **aceleração gráfica (Vulkan)** está marcada para usar a sua **Radeon 780M** [3, 10].
5. Clique no botão **Start Server** (o servidor iniciará por padrão no endereço `http://localhost:1234`).
6. Carregando o seu modelo Hermes 3 na lista superior do servidor, o seu cérebro de IA já estará ativo e pronto para receber perguntas!

---

## 🧙‍♂️ Passo 2: Rodar o Assistente de Configuração do Hermes Agent

O Hermes Agent precisa de uma configuração inicial interativa para definir as preferências e canais de comunicação (como bots do Telegram, Discord ou Slack).

Para permitir que ele converse com o seu LM Studio na máquina host através do Docker, crie a pasta de memórias e execute o assistente passando o parâmetro de ponte de rede:

1. No terminal do seu Bluefin, crie a pasta do Hermes:
   ```bash
   mkdir -p ~/.hermes
   ```

2. Execute o comando para abrir o configurador oficial:
   ```bash
   docker run -it --rm \
     --add-host=host.docker.internal:host-gateway \
     -v ~/.hermes:/opt/data \
     nousresearch/hermes-agent setup
   ```

3. Durante o assistente:
   * Quando ele perguntar o provedor de IA, selecione a opção **Custom/OpenAI-compatible**.
   * Quando ele pedir a **Base URL (URL de Origem)**, digite exatamente:  
     👉 `http://host.docker.internal:1234/v1`
   * Para a **API Key**, você pode digitar qualquer palavra (por exemplo, `lm-studio`), já que o LM Studio não exige chaves de autenticação de verdade.
   * Conclua o assistente respondendo sobre os canais de mensagens e as configurações serão salvas na pasta `~/.hermes/.env` de forma persistente.

---

## 📦 Passo 3: Criar o arquivo de Configuração (`docker-compose.yml`)

Agora vamos criar a receita do Docker Compose para subir a interface visual (Open WebUI) e o Hermes Agent integrados ao LM Studio.

1. No seu terminal, crie a pasta do projeto:
   ```bash
   mkdir -p ~/ia-local && cd ~/ia-local
   ```

2. Abra o editor de texto:
   ```bash
   nano docker-compose.yml
   ```

3. Copie o código abaixo, cole dentro do editor, salve pressionando **Ctrl + O**, depois **Enter**, e saia com **Ctrl + X**:

```yaml
version: '3.8'

services:
  # 1. Interface Visual de Chat (Open WebUI) conectada ao LM Studio
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: unless-stopped
    ports:
      - "3000:8080"
    volumes:
      - open_webui_data:/app/backend/data
    environment:
      - OPENAI_API_BASE_URL=http://host.docker.internal:1234/v1
      - OPENAI_API_KEY=lm-studio
    extra_hosts:
      - "host.docker.internal:host-gateway" # Permite ao container enxergar o LM Studio no host

  # 2. Hermes Agent (Nous Research) - Agente Autônomo
  hermes-agent:
    image: nousresearch/hermes-agent:latest
    container_name: hermes-agent
    restart: unless-stopped
    ports:
      - "8642:8642"
    volumes:
      - ~/.hermes:/opt/data
    extra_hosts:
      - "host.docker.internal:host-gateway" # Permite ao container enxergar o LM Studio no host

volumes:
  open_webui_data:
```

---

## ⚡ Passo 4: Subir a Infraestrutura

Com tudo pronto, execute apenas este comando no terminal dentro da pasta `~/ia-local` para subir os containers:

```bash
docker compose up -d
```

O Docker baixará e iniciará os dois containers. Como o processamento do modelo está sendo feito de forma nativa e otimizada pelo LM Studio via Vulkan [3], você terá respostas extremamente rápidas, menor consumo de energia e total estabilidade!

---

## 🎮 Como Interagir

* **Acessar a Interface Gráfica**: Abra o seu navegador e vá para `http://localhost:3000` para conversar visualmente com o modelo rodando no LM Studio.
* **Controlar o Hermes Agent no Terminal**: Execute o comando abaixo se quiser dar instruções diretas de terminal ao agente autônomo:
  ```bash
  docker exec -it hermes-agent hermes
  ```
* **Acesso Remoto (Telegram/Discord)**: Se configurado no assistente do Passo 2, o Hermes Agent responderá você diretamente de qualquer lugar através do chat do seu bot de mensagem!
