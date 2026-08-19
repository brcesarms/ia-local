# Guia Completo: Hermes Agent e Open WebUI alimentados pelo LM Studio (Vulkan) no Bluefin Linux

Este repositório contém o guia definitivo para configurar um ecossistema local de Inteligência Artificial de alta performance no seu **Mini PC GEEKOM A7 MAX** (equipado com Ryzen 9 7940HS, Radeon 780M e 64 GB de RAM DDR5) [3].

Em vez de lutar com a configuração complexa do AMD ROCm ou permissões de driver dentro de containers Docker [3], esta abordagem utiliza o **LM Studio** rodando nativamente na máquina host como o provedor de modelos (usando aceleração Vulkan integrada) [3, 10]. Isso garante taxas impressionantes de **20 a 23 tokens por segundo** sem complicações de configuração de GPU [3].

---

## 🛠️ Arquitetura do Projeto

1. **LM Studio (Host)**: Roda nativamente via Vulkan [3, 10], carregando o modelo (ex: **Hermes 3 8B**) e expondo uma API compatível com OpenAI na porta `1234` [3].
2. **Open WebUI (Docker)**: Interface gráfica linda para conversar com seus modelos no navegador.
3. **Hermes Agent (Docker)**: Agente autônomo da Nous Research que executa tarefas, lembra de conversas antigas e gerencia ferramentas.
4. **Ponte de Rede (`host.docker.internal`)**: Permite que os containers dentro do Docker acessem com segurança o servidor de IA rodando diretamente no seu sistema.

---

## 🚀 Passo a Passo de Instalação

### Passo 1: Configurar o LM Studio (O Cérebro)
1. Abra o **LM Studio** no seu Bluefin.
2. Baixe o modelo **Hermes 3 8B GGUF** (ou outro de sua preferência).
3. Vá na aba **Local Server** (ícone de setas duplas no menu lateral esquerdo).
4. Certifique-se de que a aceleração por hardware **Vulkan** está selecionada para usar a sua **Radeon 780M** [3, 10].
5. Clique em **Start Server** (por padrão iniciará em `http://localhost:1234`).

### Passo 2: Executar o Assistente do Hermes Agent
Para criar as configurações necessárias e apontá-las para o LM Studio através do Docker:

```bash
mkdir -p ~/.hermes
docker run -it --rm \
  --add-host=host.docker.internal:host-gateway \
  -v ~/.hermes:/opt/data \
  nousresearch/hermes-agent setup
```
* No assistente, selecione **Custom/OpenAI-compatible** e insira a Base URL: `http://host.docker.internal:1234/v1`. Para API Key, digite `lm-studio` (ou qualquer texto).

### Passo 3: Criar o Arquivo `docker-compose.yml`
Crie uma pasta de projeto e monte o arquivo de configuração única:

```bash
mkdir -p ~/ia-local && cd ~/ia-local
nano docker-compose.yml
```

Cole o seguinte código:

```yaml
version: '3.8'

services:
  # Interface Visual de Chat (Open WebUI)
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
      - "host.docker.internal:host-gateway"

  # Hermes Agent (Nous Research) - Agente Autônomo
  hermes-agent:
    image: nousresearch/hermes-agent:latest
    container_name: hermes-agent
    restart: unless-stopped
    ports:
      - "8642:8642"
    volumes:
      - ~/.hermes:/opt/data
    extra_hosts:
      - "host.docker.internal:host-gateway"

volumes:
  open_webui_data:
```

### Passo 4: Subir Todo o Sistema
Dentro de `~/ia-local`, execute o comando:

```bash
docker compose up -d
```

---

## 🎮 Acesso e Controle

* **Open WebUI**: Acesse `http://localhost:3000` no seu navegador de internet.
* **Hermes Agent (Console)**: Controle o agente diretamente no terminal com o comando:
  ```bash
  docker exec -it hermes-agent hermes
  ```
* **Bot de Mensagens**: Se configurado no assistente, o Hermes responderá você de forma autônoma pelo Telegram, Discord ou Slack!
