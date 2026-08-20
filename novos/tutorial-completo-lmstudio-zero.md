# Guia Absoluto: Configuração do LM Studio Zero-to-Hero no Linux Bluefin

Este é o guia definitivo para configurar o seu ecossistema de Inteligência Artificial local do absoluto zero no seu **Mini PC GEEKOM A7 MAX (Ryzen 9, 64 GB de RAM, Radeon 780M)** rodando **Linux Bluefin**. 

Como o Bluefin é um sistema operacional atômico e imutável, este tutorial utiliza caminhos isolados em nível de usuário (`~/.local` e `~/.config`) e containers **Docker** para garantir que o seu sistema permaneça intacto, limpo e extremamente performático.

Ao final deste guia, você terá o **LM Studio** e a interface web **Open WebUI** iniciando de forma 100% automática sempre que ligar seu Mini PC, prontos para uso local e privado, integrados também com o **Obsidian**.

---

## 🏗️ Requisitos do Sistema e Estrutura Inicial
Antes de começar, vamos criar a estrutura de diretórios necessária no seu espaço de usuário para manter tudo organizado. Abra o terminal e rode:

```bash
mkdir -p ~/Applications
mkdir -p ~/.local/bin
mkdir -p ~/.config/autostart
mkdir -p ~/.config/systemd/user
```

*   `~/Applications`: Guardará o arquivo executável (AppImage) do LM Studio.
*   `~/.local/bin`: Guardará scripts utilitários pessoais.
*   `~/.config/autostart`: Responsável por iniciar aplicativos gráficos no login.
*   `~/.config/systemd/user`: Responsável por gerenciar serviços de segundo plano a nível de usuário.

---

## 📥 Passo 1: Baixar e Instalar o LM Studio
1. Acesse o terminal do seu Bluefin e baixe a versão mais recente do LM Studio para Linux (AppImage) diretamente na pasta criada:
   ```bash
   curl -L -o ~/Applications/LM_Studio.AppImage "https://lmstudio.ai/download/linux/appimage"
   ```
2. Conceda permissão de execução para o arquivo:
   ```bash
   chmod +x ~/Applications/LM_Studio.AppImage
   ```

*(Opcional)* Para testar se o programa abre sem problemas no seu ambiente visual, execute:
```bash
~/Applications/LM_Studio.AppImage
```
Feche a interface gráfica após verificar que o aplicativo iniciou corretamente.

---

## 🐳 Passo 2: Criar e Configurar o Contêiner do Open WebUI no Docker (Do Zero)
Como o Linux Bluefin já traz o Docker pré-instalado e configurado, não precisamos de nenhuma instalação manual de gerenciador de containers. Vamos criar o container do **Open WebUI** configurado para iniciar automaticamente com o sistema.

1. No terminal, execute o comando abaixo para baixar e rodar o Open WebUI do zero:
   ```bash
   docker run -d -p 3000:8080 \
     --restart unless-stopped \
     --add-host=host.docker.internal:host-gateway \
     -v open-webui:/app/backend/data \
     --name open-webui \
     ghcr.io/open-webui/open-webui:main
   ```
   
   **O que este comando faz?**
   *   `-d`: Roda o contêiner em segundo plano (detached mode).
   *   `-p 3000:8080`: Mapeia a porta `3000` do seu computador para a porta interna do contêiner.
   *   `--restart unless-stopped`: Garante que, se você reiniciar o computador ou o serviço do Docker cair, o contêiner iniciará sozinho no boot.
   *   `--add-host=host.docker.internal:host-gateway`: Cria uma rota interna inteligente no contêiner para que ele enxergue o LM Studio (que roda fora do Docker) usando o endereço `host.docker.internal`.
   *   `-v open-webui:/app/backend/data`: Persiste todos os seus chats, logins e configurações em um volume seguro para você não perder nada quando reiniciar.

2. Teste o acesso ao Open WebUI:
   *   Abra o seu navegador e acesse: `http://localhost:3000`
   *   Crie sua conta de administrador (o primeiro cadastro na máquina local se torna o administrador).

---

## 🧠 Passo 3: Baixar os Modelos de IA Ideais no LM Studio
Graças aos incríveis **64 GB de RAM** do seu GEEKOM A7 MAX, você pode carregar modelos pesados e altamente inteligentes locais com facilidade. Baixaremos dois tipos de modelos: um de **Chat** e outro de **Embedding** (usado para indexar suas notas no Obsidian).

1. Abra o **LM Studio** manualmente (`~/Applications/LM_Studio.AppImage`).
2. Clique no ícone de lupa (**Search Models**) no menu lateral esquerdo.
3. **Baixe o modelo de Chat:** Procure por `Qwen 2.5` (ou `Qwen 2.5 Instruct`). Baixe a versão recomendada (como o `Qwen 2.5 14B` ou até o potente `32B` em formato GGUF).
4. **Baixe o modelo de Embedding:** Procure por `Nomic Embed Text 1.5` (ou o `BGE-M3`) e faça o download dele.
5. Acesse o menu **Developer / Local Server** (ícone de código `<>` na barra lateral esquerda):
   *   Ative a opção **CORS** (Cross-Origin Resource Sharing). Isso é obrigatório para as integrações externas funcionarem.
   *   Selecione o seu modelo de Chat no seletor de modelos no topo da tela para carregá-lo na RAM do seu processador Ryzen 9.

---

## ⚡ Passo 4: Autostart Total no Boot do Sistema (Escolha sua Rota)

Decida agora como você prefere utilizar o LM Studio no dia a dia. Você pode escolher a **Rota A (Interface Gráfica)** ou a **Rota B (Totalmente Invisível/Headless)**.

---

### 🟢 ROTA A: Inicialização Automática com Interface Gráfica (GUI)
Ideal para quem prefere acompanhar o consumo de hardware, gerenciar e baixar novos modelos de forma visual sem precisar de terminal.

#### 1. Configurar o arquivo de autostart do Linux
Criaremos um arquivo de inicialização para o seu usuário. No terminal, execute:
```bash
nano ~/.config/autostart/lm-studio.desktop
```
Cole o conteúdo a seguir dentro do arquivo:
```ini
[Desktop Entry]
Type=Application
Name=LM Studio
Comment=Start LM Studio on system login
Exec=/home/seu-usuario/Applications/LM_Studio.AppImage --minimized
Icon=lm-studio
Terminal=false
Categories=Utility;Development;
X-GNOME-Autostart-enabled=true
```
*(Certifique-se de substituir `seu-usuario` pelo seu nome de usuário real no Linux. Você pode descobrir seu usuário rodando `whoami` no terminal).*

Dê permissão de execução ao atalho gráfico:
```bash
chmod +x ~/.config/autostart/lm-studio.desktop
```

#### 2. Configurações internas do LM Studio para iniciar o servidor
1. Com o LM Studio aberto, clique em **Settings** (ícone de engrenagem no canto inferior esquerdo).
2. Ative a caixa **"Start Local Server on Application Start"** (Iniciar servidor local ao iniciar o aplicativo). Isso garante que o servidor escutando na porta `1234` seja ligado sem cliques manuais.
3. Ative a opção **"Auto-load model on application startup"** para que o seu modelo Qwen carregue sozinho na RAM sempre que o aplicativo subir.

---

### 🔌 ROTA B: Inicialização Automática sem Interface Gráfica (Headless/CLI)
Ideal para quem busca a máxima performance do hardware (AMD Radeon 780M), economizando memória que seria usada renderizando a interface gráfica clássica.

#### 1. Configurar e expor o binário `lms` do LM Studio
Por padrão, o LM Studio disponibiliza um executável em linha de comando chamado `lms`. Vamos criar um script para acionar o servidor e carregar os modelos de forma sequencial na memória.

Crie o arquivo de script:
```bash
nano ~/.local/bin/autostart-lms.sh
```
Cole as instruções abaixo (ajustando o ID do modelo para o nome exato que você baixou. Você pode verificar os IDs dos modelos que baixou rodando `~/Applications/LM_Studio.AppImage lms ls` no terminal):
```bash
#!/bin/bash
# Aguarda 5 segundos para o sistema estabilizar a rede e conexões
sleep 5

# Inicializa o daemon do servidor local do LM Studio
lms server start

# Carrega os modelos de Chat e de Embedding em sequência
lms load qwen-2.5-14b
lms load nomic-embed-text
```
Salve o arquivo e aplique a permissão de execução:
```bash
chmod +x ~/.local/bin/autostart-lms.sh
```

#### 2. Criar o serviço de Usuário no Systemd
Como o Linux Bluefin é imutável, nunca editaremos diretórios de sistema como `/etc`. Criaremos um serviço exclusivo para o seu usuário:
```bash
nano ~/.config/systemd/user/lmstudio.service
```
Cole o seguinte conteúdo:
```ini
[Unit]
Description=LM Studio Headless Server Daemon
After=network.target

[Service]
Type=simple
ExecStart=%h/.local/bin/autostart-lms.sh
Restart=on-failure
RestartSec=10

[Install]
WantedBy=default.target
```
*(O systemd substitui `%h` automaticamente pela sua pasta Home absoluta).*

#### 3. Habilitar e Iniciar o Serviço de Usuário
Execute estes comandos para recarregar o sistema e ativar o boot do LM Studio:
```bash
systemctl --user daemon-reload
systemctl --user enable lmstudio.service
systemctl --user start lmstudio.service
```

#### 4. Ativar Linger (Essencial para Sistemas de Inicialização de Usuário)
Para garantir que o LM Studio headless inicialize **assim que a máquina liga na tomada**, antes de você digitar a senha de login:
```bash
loginctl enable-linger $USER
```

---

## 📝 Passo 5: Conectando com o Obsidian Copilot (Do Zero)
Agora que seu ecossistema local do LM Studio + Docker está ligando de forma automatizada, vamos conectar suas notas privadas do Obsidian para utilizá-las como base de conhecimento!

1. Abra o **Obsidian** e acesse: **Settings (Engrenagem) > Community Plugins**.
2. Ative os plugins de comunidade (caso estejam desativados) e clique em **Browse**.
3. Pesquise por **Copilot** (desenvolvido por *Logan Yang*) e clique em **Install**, seguido de **Enable**.
4. Acesse as configurações do plugin: **Settings > Copilot (na seção lateral esquerda)**.
5. Na aba **Model**, clique em **Add Custom Model**:
   *   **Provider:** Selecione `LM Studio`.
   *   **Model Name:** Cole o ID exato do seu modelo de Chat ativo (ex: `qwen-2.5-14b`).
   *   **Base URL:** Insira `http://localhost:1234/v1`.
   *   Ative o botão **CORS** se aparecer na tela do modelo adicionado.
6. Clique em **Verify** e depois em **Add Model**.
7. Na aba **QA Settings**:
   *   Defina o **Embedding Provider** como `LM Studio`.
   *   Defina o **Embedding Model** como `nomic-embed-text` (ou o ID do modelo que você baixou).
   *   Clique em **Build Index** para que a IA comece a ler e indexar o seu cofre de notas em tempo de execução 100% privado.

---

## ✅ Validação Final
Sempre que ligar seu computador, sem que você precise digitar um comando sequer no terminal:
*   Acesse `http://localhost:3000` no navegador para conversar com sua IA local na web.
*   Abra o **Obsidian** e use a barra lateral do **Copilot** para interagir diretamente com suas notas privadas de forma segura e offline.
*   No terminal, confira o status do serviço headless rodando:
    ```bash
    systemctl --user status lmstudio.service
    ```
