---
title: "Guia Passo a Passo: Obsidian + IA Local (Ollama + Copilot)"
date_created: 2025-08-17
tags:
  - tutorial/obsidian
  - tutorial/ia-local
  - ferramentas/ollama
  - ferramentas/copilot
---

# Guia Passo a Passo: Obsidian + IA Local (Ollama + Copilot)

> [!info] Resumo
> Guia prático para configurar um assistente de IA executado **100% localmente** no seu computador, garantindo privacidade absoluta para as suas notas — nenhum dado é enviado para a nuvem.

---

## 📦 Passo 1: Instalando o Ollama no Toolbx

> [!tip] Por que usar Toolbx?
> O **Toolbx** mantém seu sistema operacional base limpo, isolando o ambiente de IA em um container seguro que compartilha os recursos de rede e acesso à GPU do hospedeiro automaticamente.

1. Abra o terminal do seu Linux.
2. Crie e acesse um container dedicado para IA:

```bash
toolbox create -c ai-env
toolbox enter -c ai-env
```

3. Instale o Ollama executando o script oficial dentro do container:

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

4. Inicie o servidor do Ollama manualmente (containers Toolbx normalmente não utilizam o `systemd` por padrão):

```bash
ollama serve
```

> [!warning] Mantenha este terminal aberto!
> O serviço do Ollama continua rodando em segundo plano na porta padrão `11434`. Fechar esta janela encerrará o serviço.

---

## 🧠 Passo 2: Escolhendo e Baixando as Melhores IAs Locais

Para que o Obsidian consiga interagir com suas notas de forma inteligente, você precisará de **dois tipos de modelos**:

- **Chat/Automação** — entenda comandos de chamada de função (Function Calling)
- **Embeddings** — execute a busca semântica no vault

> [!info] Abra um novo terminal
> Entre no mesmo container (`toolbox enter -c ai-env`) para executar os comandos abaixo.

### 🗣️ Modelo de Chat e Raciocínio: `qwen2.5:7b`

A família Qwen oferece excelente suporte em português e ótima precisão para chamadas de função, permitindo que a IA manipule notas se você autorizar.

```bash
ollama run qwen2.5:7b
```

> [!tip] Hardware modesto?
> Você pode optar pela versão mais leve: `ollama run qwen3.5:4b` ou `qwen2.5:3b`.

### 🔍 Modelo de Embeddings: `nomic-embed-text`

Essencial para indexar seu vault localmente e possibilitar a busca semântica em todo o seu repositório de notas.

```bash
ollama pull nomic-embed-text
```

---

## 🔌 Passo 3: Instalando e Configurando o Plugin Copilot no Obsidian

O plugin **Copilot** é um dos melhores assistentes gerais para Obsidian, permitindo chat, geração inline e busca semântica local.

### 📥 1. Instalação do Plugin

1. Abra o **Obsidian**.
2. Vá em **Settings → Community Plugins**.
3. Clique em **Browse** e busque por **Copilot**.
4. Clique em **Install** e depois em **Enable**.

### ⚙️ 2. Configurações de Conexão com o Ollama

1. Abra a aba de configurações do **Copilot** na barra lateral de configurações do Obsidian.
2. Altere o campo **Provider** para **Ollama**.
3. Confirme que a **Ollama API URL** está definida como:

```text
http://localhost:11434
```

4. No campo **Active Model**, selecione ou digite exatamente o modelo que você baixou (ex: `qwen2.5:7b`).

### 🗂️ 3. Configurando a Busca Semântica (Embeddings)

Para permitir que o Copilot vasculhe todas as suas notas locais em busca de respostas:

1. Ainda nas configurações do plugin **Copilot**, role a tela até a seção **Embedding Settings** ou **Local Index**.
2. Defina o provedor como **Ollama**.
3. Selecione o modelo **`nomic-embed-text`**.

---

## 💻 Passo 4: Como Usar o Chat com suas Notas ("Chat with Vault")

O recurso de conversar com sua base de conhecimento inteira fica no painel lateral de chat:

1. Clique no **ícone do Copilot na barra lateral direita** do Obsidian para abrir a janela de chat.
2. No cabeçalho ou rodapé do painel de conversa, você verá um seletor de modo (onde está escrito "Chat").
3. Clique no seletor e mude para **"Vault QA"** ou **"Long-Term Memory"** (esses são os nomes utilizados pelo Copilot para o recurso de conversar com o Vault).
4. O Obsidian solicitará que você faça a primeira indexação de suas notas. Clique em **"Index Vault"**.
5. Aguarde a finalização do processo (o tempo varia conforme o tamanho do seu vault).

> [!tip] Pronto!
> Agora você tem um cérebro digital superpotente e completamente offline trabalhando ao seu lado. 🚀

---

## 🔗 Notas Relacionadas

- [Formatar Vault](format_vault.py) — Script Python para padronizar a formatação do vault.
