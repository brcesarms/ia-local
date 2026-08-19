# Guia de Configuração do AMD ROCm para Radeon 780M no Bluefin Linux

Este guia detalhado foi projetado para ajudá-lo a configurar a aceleração de Inteligência Artificial por hardware (GPU) utilizando o **AMD ROCm** e o **PyTorch** no seu mini PC **GEEKOM A7 MAX** rodando o sistema operacional **Bluefin Linux**.

Como a BIOS de fábrica do GEEKOM A7 MAX possui menus bloqueados que impedem a alteração manual do tamanho da VRAM (memória dedicada da GPU), este tutorial ensina a contornar essa restrição diretamente no sistema de arquivos do Linux utilizando a memória GTT (Graphics Translation Table) de forma dinâmica, além de isolar todo o ambiente técnico dentro de um container seguro com o **Distrobox**.

---

## 💻 Especificações do Sistema Alvo
*   **Equipamento:** Mini PC GEEKOM A7 MAX (Edição AI)
*   **Processador:** AMD Ryzen 9 7940HS
*   **Memória RAM:** 64 GB DDR5
*   **Placa de Vídeo:** AMD Radeon 780M (Integrada / iGPU - arquitetura `gfx1103`)
*   **Sistema Operacional:** Bluefin Linux (baseado em Fedora CoreOS / Silverblue)

---

## 🛠️ Passo a Passo da Configuração

### Passo 1: Otimizar a Alocação de Memória da GPU (No Sistema Host)
Como a Radeon 780M compartilha a memória RAM física com o processador, o driver de vídeo do Linux (`amdgpu`) limita por padrão o uso dinâmico (GTT) à metade da sua RAM total. No seu caso, com 64 GB de RAM, vamos otimizar essa alocação para que a GPU possa utilizar dinamicamente até 45 GB estáveis para tarefas pesadas de Inteligência Artificial.

1. Abra o seu terminal principal do Bluefin Linux (**Ptyxis** ou **Terminal**).
2. Crie e edite um arquivo de configuração para o driver AMD executando o seguinte comando:
   ```bash
   sudo nano /etc/modprobe.d/increase_amd_memory.conf
   ```
3. Copie e cole as seguintes linhas dentro do arquivo aberto no terminal:
   ```text
   # Aumenta o tamanho do pool de memória compartilhada para a iGPU
   options amdgpu gttsize=45000
   options ttm pages_limit=11250000
   options ttm page_pool_size=11250000
   ```
4. Salve e feche o arquivo pressionando as teclas `Ctrl + O`, seguidas de `Enter`, e depois feche com `Ctrl + X`.
5. **Reinicie o seu Mini PC** para aplicar essas otimizações no Kernel antes de prosseguir.

---

### Passo 2: Criar o Container Isolado via Distrobox (Ubuntu 22.04)
Para manter o seu Bluefin Linux limpo e livre de pacotes experimentais, utilizaremos o **Distrobox** para criar um espaço de desenvolvimento seguro com base no **Ubuntu 22.04 LTS** (que é o sistema de arquivos mais compatível e estável para as ferramentas oficiais do ROCm).

Também isolaremos a pasta do usuário (`--home`) para garantir que os arquivos de cache de modelos de linguagem e difusão de imagens não se misturem à sua pasta principal de usuário.

1. No terminal do Bluefin, execute o comando de criação:
   ```bash
   distrobox create --name igpu --home ~/podhome/igpu --image ubuntu:22.04
   ```
   *(Pressione `y` e dê `Enter` caso o sistema pergunte se deseja baixar a imagem de container necessária).*

---

### Passo 3: Acessar a "Sala de Testes" do Container
Uma vez criado o container, entre no ambiente digitando o seguinte comando:
```bash
distrobox enter igpu
```
*(Você notará que o nome no início da linha de comandos mudará para `igpu`, indicando que você agora está operando com segurança dentro do Ubuntu isolado).*

---

### Passo 4: Instalar as Dependências Básicas de Sistema
Dentro do container, atualize a lista de repositórios e instale ferramentas essenciais de desenvolvimento e monitoramento de desempenho (como o `radeontop`, que permite visualizar a atividade da iGPU Radeon 780M):

```bash
sudo apt update && sudo apt install git python3-venv radeontop -y
```

---

### Passo 5: Criar o Ambiente Virtual Python
Criaremos um espaço de pacotes Python isolado dentro da pasta de usuário do próprio container para evitar conflitos de versões de bibliotecas de Machine Learning:

```bash
cd ~
python3 -m venv pyenv
```

---

### Passo 6: Instalar o PyTorch com Suporte ao AMD ROCm
Instale a versão especial do PyTorch compilada com os recursos para a plataforma de processamento em GPU da AMD:

```bash
./pyenv/bin/pip install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/rocm5.7
```
*(Esse processo pode levar alguns minutos devido ao tamanho dos pacotes de aceleração de hardware. Aguarde até a conclusão).*

---

### Passo 7: O Truque de Ouro - Enganar o ROCm (Override de GPU)
A AMD não fornece suporte nativo automatizado para placas integradas de notebooks ou mini PCs no ecossistema ROCm. Sua placa Radeon 780M é identificada internamente pela arquitetura `gfx1103`.

Para que as aplicações não travem ou façam fallback para a lentidão do processador (CPU), vamos forçar o ROCm a fingir que a sua GPU integrada é uma placa gráfica de desktop dedicada totalmente suportada (no caso, a arquitetura RDNA3 `gfx1100`).

Aplique esse truque de forma permanente no arquivo de inicialização do terminal do seu container:

```bash
echo 'export HSA_OVERRIDE_GFX_VERSION=11.0.0' >> ~/.bashrc
source ~/.bashrc
```

---

## 🧪 Validando a Instalação

Agora, vamos fazer o teste definitivo para garantir que o ambiente consegue ver o chip integrado Radeon 780M.

1. Abra o terminal Python dentro do seu container:
   ```bash
   ./pyenv/bin/python
   ```
2. Digite ou cole as seguintes duas linhas de código, apertando `Enter` após cada uma:
   ```python
   import torch
   print(torch.cuda.is_available())
   ```

Se o terminal imprimir **`True`** na tela, parabéns! Seu ambiente está perfeitamente integrado e utilizando a Radeon 780M de forma nativa e acelerada.

*(Para sair da tela do Python, digite `exit()` e dê `Enter`).*

---

## 🤖 Como Usar no Mundo Real

### 1. Rodando o Ollama com Aceleração por GPU
Para rodar Inteligência Artificial local de texto (Estilo ChatGPT) usando o Ollama totalmente acelerado por hardware, inicie o servidor passando a variável de override:

```bash
HSA_OVERRIDE_GFX_VERSION=11.0.0 ollama serve
```
Isso descarrega os tensores de peso do modelo diretamente na memória RAM mapeada para a GPU Radeon 780M, entregando velocidades de resposta até 6x mais rápidas do que o processamento em CPU.

### 2. Rodando Geradores de Imagem (Stable Diffusion / ComfyUI)
Se você for clonar e utilizar o **ComfyUI** dentro do seu container virtual, execute o aplicativo limitando o consumo de VRAM e desativando o VAE pesado da GPU para evitar travamentos por limite físico de buffer:

```bash
HSA_OVERRIDE_GFX_VERSION=11.0.0 ../pyenv/bin/python main.py --novram --cpu-vae
```

---

## ⚠️ O que Esperar: O Efeito de "Aquecimento" (Warm-up)
Um comportamento comum e amplamente reportado por usuários de hardware móvel AMD (como laptops Framework e Mini PCs) no Linux é o "efeito de aquecimento":
*   Ao processar a primeira imagem ou carregar o primeiro modelo pesado de linguagem, **a tela do computador pode piscar ou ficar preta por cerca de 1 a 2 segundos**.
*   O driver de vídeo do sistema operacional (`amdgpu`) detectará o pico de uso agressivo de memória e pode reiniciar a iGPU (visível nos logs do kernel com comandos `dmesg`).
*   Se o seu programa travar ou fechar durante esse primeiro reset de segurança, não se preocupe: **apenas reinicie o programa ou comando de IA**. Na segunda tentativa em diante, o pipeline de computação do ROCm funcionará de forma estável, contínua e extremamente veloz.

---
*Este documento é de domínio público e foi estruturado a partir de soluções criadas coletivamente pelas comunidades Framework Laptop e Project Bluefin.*
