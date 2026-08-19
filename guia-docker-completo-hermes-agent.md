# GEEKOM A7 MAX - Local AI Setup (Bluefin Linux + Docker ROCm)

Este repositório contém o guia definitivo e a configuração automatizada para rodar um ecossistema completo de **Inteligência Artificial local** de forma acelerada por hardware no **Mini PC GEEKOM A7 MAX (Edição AI)**, utilizando o sistema operacional **Bluefin Linux**.

A arquitetura utiliza o **Docker Compose** para orquestrar e integrar as imagens oficiais prontas, garantindo uma instalação limpa, sem a necessidade de compilar pacotes ou gerenciar dependências complexas do sistema operacional.

---

## 💻 Especificações do Setup Alvo
Este guia foi otimizado e testado para a seguinte configuração:
* **Equipamento:** Mini PC GEEKOM A7 MAX (Edição AI)
* **Processador:** AMD Ryzen 9 7940HS (com IA Ryzen AI NPU dedicada)
* **Placa de Vídeo:** AMD Radeon 780M integrada (arquitetura `gfx1103`)
* **Memória RAM:** 64 GB DDR5 (perfeita para alocação de vídeo compartilhada massiva)
* **Sistema Operacional:** Linux Bluefin (Universal Blue) — um sistema atômico e imutável de próxima geração.

---

## 📦 Componentes do Ecossistema
A receita automatizada do Docker Compose inicia três serviços principais integrados:
1. **Ollama (com aceleração AMD ROCm):** O motor de inferência local, configurado para emular a iGPU Radeon 780M (`gfx1103`) como uma dGPU RDNA3 suportada (`gfx1100`) usando a variável de ambiente `HSA_OVERRIDE_GFX_VERSION=11.0.0`.
2. **Open WebUI:** Uma interface de chat rica e bonita integrada ao navegador, com visual e recursos similares ao ChatGPT, permitindo baixar e gerenciar modelos de forma visual.
3. **Hermes Agent (Nous Research):** Um agente autônomo de inteligência artificial de última geração, capaz de interagir pelo terminal ou integrar-se a canais de mensagem (como Telegram, Discord ou Slack), mantendo memórias de longo prazo e aprendendo novas habilidades de forma persistente.

---

## 🛠️ Requisitos Iniciais de Sistema

Como o GEEKOM A7 MAX possui uma BIOS original com opções de alocação de vídeo de fábrica travadas, o sistema operacional gerencia a memória de forma dinâmica através do driver open-source da AMD. 

Com os seus **64 GB de RAM**, a placa de vídeo Radeon 780M integrada é capaz de emprestar dinamicamente **até 32 GB de memória gráfica** sob demanda. Para garantir que o motor do Docker tenha permissão total de acesso gráfico direto à aceleração por hardware, certifique-se de que o Docker está ativado para iniciar junto com o sistema host:

```bash
sudo systemctl enable docker
```

---

## 🚀 Passo a Passo de Instalação

### Passo 1: Criar a pasta do projeto
Abra o terminal (**Ptyxis** ou **Terminal**) no seu Bluefin e crie uma pasta organizada para hospedar os arquivos de configuração:

```bash
mkdir -p ~/ia-local && cd ~/ia-local
```

### Passo 2: Criar o arquivo de configuração única (`docker-compose.yml`)
1. No terminal, abra o editor de texto integrado rodando:
   ```bash
   nano docker-compose.yml
   ```
2. Copie todo o código do bloco abaixo e cole no terminal:

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
      - HSA_OVERRIDE_GFX_VERSION=11.0.0 # Override crucial para emular a Radeon 780M
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

3. Salve e saia do editor:
   * Pressione **Ctrl + O** e aperte **Enter** para confirmar a gravação.
   * Pressione **Ctrl + X** para fechar o editor.

---

### Passo 3: Configurar o Hermes Agent (Primeira Execução)
O **Hermes Agent** exige um assistente interativo inicial para carregar suas memórias e chaves de API preferidas.

1. Crie o diretório de dados persistentes na sua máquina host:
   ```bash
   mkdir -p ~/.hermes
   ```
2. Execute o assistente de configuração oficial em modo interativo rodando:
   ```bash
   docker run -it --rm -v ~/.hermes:/opt/data nousresearch/hermes-agent setup
   ```
3. Responda às perguntas simples do assistente na tela (selecione o provedor do Ollama local ou serviços em nuvem como OpenRouter, adicione suas chaves de API e defina se deseja habilitar canais adicionais de comunicação como bots de Telegram). Assim que concluir, o assistente salvará as configurações de forma segura em `~/.hermes/.env` e fechará o container temporário.

---

### Passo 4: Subir todo o Ecossistema
Com o arquivo de receita salvo e a configuração do seu agente finalizada, suba todo o stack de containers em segundo plano rodando:

```bash
docker compose up -d
```

O Docker fará o download e a inicialização de todas as imagens oficiais necessárias automaticamente do Docker Hub. Isso pode levar alguns minutos na primeira execução.

---

## 🎮 Como Usar o seu Setup de IA

### 1. Interface Web do Chat (Open WebUI)
* Abra seu navegador de internet e acesse: **`http://localhost:3000`**
* Crie o seu primeiro cadastro de usuário administrador local (todas as conversas ficam salvas localmente em seu próprio hardware).
* Na aba de configurações, faça o download do modelo **Hermes 3** (por exemplo, `hermes3:8b`) digitando o identificador do modelo de forma visual na aba de gerenciamento do Ollama.

### 2. Conversar ou Controlar o Hermes Agent
O Hermes Agent estará rodando de forma persistente em segundo plano na porta `8642`. Para interagir com ele de forma autônoma diretamente no terminal do seu computador, execute:

```bash
docker exec -it hermes-agent hermes
```

---

## ⚠️ Comportamento de "Warm-up" (Aviso para Iniciantes)
Ao rodar tarefas de Inteligência Artificial pesadas pela primeira vez usando a placa gráfica integrada do mini PC, **a tela de exibição do computador pode piscar rapidamente ou ficar escura por cerca de um segundo**. 

Isso é um comportamento conhecido de "warm-up" (aquecimento e reinicialização de segurança da GPU) reportado pela comunidade. Se o programa fechar ou apresentar uma falha na primeira tentativa, apenas aguarde o reset de segurança da GPU e execute-o novamente. A partir da segunda execução em diante, o processamento ocorre com excelente estabilidade e rapidez.

---

## 🔄 Manutenção e Atualização de Aplicativos
Mantenha todo o seu ecossistema de ferramentas atualizado com as últimas versões disponibilizadas pela AMD, Open WebUI e Nous Research sem perder nenhuma conversa ou memória salva:

```bash
cd ~/ia-local
docker compose pull
docker compose up -d
```

---
*Criado com base nos relatórios de comunidade e documentação técnica oficial de suporte do Project Bluefin e do AMD ROCm.*
