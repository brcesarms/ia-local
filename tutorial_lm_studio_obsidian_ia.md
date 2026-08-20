---
title: "Guia Passo a Passo: Obsidian + LM Studio (LLM Local)"
tags:
  - obsidian
  - lm-studio
  - ia-local
source: https://shikiyura.com/2026/03/obsidian-lm-studio-integration/
---


### Guia Passo a Passo: Obsidian + LM Studio (LLM Local)

[!info] Resumo
Guia prático para integrar o **Obsidian** ao **LM Studio**, permitindo que você conecte um assistente de inteligência artificial que roda **100% localmente** no seu próprio computador. Isso garante privacidade absoluta para as suas notas e pensamentos — nenhum dado é enviado para servidores de terceiros ou nuvens externas.

---

#### 📦 Passo 1: Instalando o Plugin Copilot no Obsidian

[!tip] Requisitos de Preparação
Certifique-se de que o **Obsidian** já esteja instalado em seu sistema operacional. Caso contrário, faça o download e realize a instalação acessando a página oficial: [obsidian.md](https://obsidian.md/).

1. Abra o **Obsidian** no seu computador.
2. Acesse as configurações clicando no ícone de engrenagem no canto inferior esquerdo (**Settings**).
3. Vá para a seção de **Community Plugins** (Plugins da Comunidade).
4. Caso ainda não tenha habilitado, ative os plugins de terceiros clicando no botão para habilitar.
5. Clique em **Browse** (Procurar) e pesquise por **Copilot**.
6. Clique no plugin Copilot nos resultados de busca, selecione **Install** (Instalar) e depois **Enable** (Ativar).

---

#### 🖥️ Passo 2: Configurando o Servidor Local no LM Studio

[!info] O que é o LM Studio?
O **LM Studio** é um software desktop que permite buscar, baixar e executar modelos de linguagem (LLMs) diretamente no seu hardware de forma simples, fornecendo um servidor local que é totalmente compatível com o formato de API da OpenAI.

1. Abra o aplicativo do **LM Studio** no seu sistema.
2. No menu lateral esquerdo, clique no segundo ícone de cima para baixo para acessar a aba **Developer** (Desenvolvedor).
3. No painel de controle do servidor de desenvolvimento, certifique-se de que o status indica **Status: Running** (Status: Em Execução).
   - [!tip] Caso o status indique **Stopped** (Parado), clique no botão deslizante (switch) ao lado para mudar o status para **Running**.
4. No menu superior da tela, certifique-se de carregar (Load) o modelo local que você deseja utilizar para interagir com suas notas.
5. Copie o endereço de conexão listado no campo **Reachable at:** (por exemplo, `http://localhost:1234`). Ele será usado para a integração.

---

#### ⚙️ Passo 3: Configurando a Conexão do LM Studio no Copilot

Agora, faremos a ponte entre o Obsidian e o LM Studio informando o endereço do servidor para o plugin Copilot.

1. Abra novamente o **Obsidian** e acesse as configurações do plugin **Copilot** (disponível na barra lateral de configurações).
2. Na parte superior das configurações do Copilot, vá para a aba **Model** (Modelo).
3. Clique no botão **Add Model** (Adicionar Modelo) para inserir o perfil do LM Studio.
4. Preencha as informações do formulário exatamente como especificado abaixo:
   - **Model Name**: Insira um nome de identificação de sua escolha para o modelo (ex: `lms-gpt20b`).
   - **Provider**: Selecione **LM Studio** na lista de opções (dropdown).
   - **Base URL**: Cole o endereço que você copiou do painel do LM Studio (no passo anterior), adicionando obrigatoriamente `/v1` ao final. Por exemplo: `http://localhost:1234/v1`.
   - **Model Capability**: Selecione a capacidade de processamento **Reasoning** (Raciocínio).
5. Clique no botão **Test** (Testar). Se a conexão estiver ativa e configurada corretamente, um símbolo de verificação verde (checkmark) aparecerá ao lado, validando o teste com sucesso.

---

#### 💬 Passo 4: Consultando Suas Notas Diretamente no Chat (Chat com Notas)

[!tip] Chega de copiar e colar!
Anteriormente, o processo exigia copiar notas longas e colá-las em prompts, gerando falhas contextuais ao longo das interações. Usando a integração nativa, o plugin Copilot envia a nota diretamente de forma otimizada para o LLM local.

1. No painel lateral direito do Obsidian, clique no **ícone do Copilot** para abrir a barra lateral de chat.
2. Na área do cabeçalho do painel de chat, use o seletor de modelos para selecionar o perfil que você criou (ele exibirá o nome que você definiu no campo *Model Name*, por exemplo, `lms-gpt20b`).
3. Vá até a caixa de texto de entrada de mensagens e clique no botão **"@"** (ou digite `@` diretamente).
4. No menu pop-up, selecione **Notes** (Notas).
5. Escolha no seletor a nota específica do seu vault que você deseja que a inteligência artificial consulte.
6. Digite sua pergunta, dúvida ou comando relacionado ao documento e envie. O LLM local lerá a nota do Obsidian e formulará a resposta com base apenas no conteúdo local de forma privada e segura!

---

#### 🔗 Notas Relacionadas
*  [Guia Ollama + Copilot](exemplo_tutorial_lm_studio_obsidian_ia.md) — Guia passo a passo alternativo para configuração de inteligência artificial local usando Ollama e Toolbx no Linux.
