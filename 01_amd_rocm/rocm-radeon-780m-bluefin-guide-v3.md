# Guia Definitivo: AMD ROCm para Radeon 780M no Linux Bluefin (Via Docker - Super Fácil)

Este guia foi criado especialmente para o mini PC **GEEKOM A7 MAX (Edição AI)** equipado com o processador **AMD Ryzen 9 7940HS** e gráficos integrados **AMD Radeon 780M (arquitetura gfx1103)**, rodando o sistema operacional **Linux Bluefin**.

Se você é leigo em informática, este guia é perfeito para você. Em vez de instalar dezenas de pacotes, compilar códigos ou criar ambientes virtuais complexos, nós vamos usar **imagens prontas do Docker Hub** mantidas diretamente pela própria **AMD** e pela comunidade do **Ollama**. É a forma mais rápida, segura e à prova de falhas!

---

## ⚙️ Passo 1: Preparação de Memória (No seu Sistema Bluefin)

Como a BIOS do seu GEEKOM A7 MAX possui limitações de configuração manual para VRAM, nós faremos um ajuste simples diretamente no Linux Bluefin. Isso garantirá que a sua placa Radeon 780M consiga emprestar até **45 GB** da sua memória RAM total (que é de 64 GB) para rodar modelos de IA gigantes sem travamentos.

1. Abra o seu **Terminal** no Bluefin.
2. Copie e cole o comando abaixo e aperte **Enter** (o sistema pedirá a sua senha):
   ```bash
   sudo nano /etc/modprobe.d/increase_amd_memory.conf
   ```
3. Uma tela de texto em branco vai se abrir. Cole as seguintes linhas exatamente como estão:
   ```text
   # Libera mais memória RAM compartilhada para a placa de vídeo integrada
   options amdgpu gttsize=45000
   options ttm pages_limit=11250000
   options ttm page_pool_size=11250000
   ```
4. Para salvar e fechar: aperte **Ctrl + O**, depois **Enter** para confirmar, e por fim **Ctrl + X** para sair do editor.
5. **Reinicie o seu mini PC** para que esta alteração de memória passe a valer.

---

## 🚀 Passo 2: Rodar o PyTorch (ROCm da AMD) com Apenas 1 Comando

O **Docker** já vem instalado e pronto para uso no Bluefin Linux. A própria AMD disponibiliza uma imagem pronta chamada `rocm/pytorch` no Docker Hub que já contém todo o sistema ROCm e o PyTorch totalmente configurados e testados. 

Para baixar a imagem e entrar nesse ambiente pronto com a sua Radeon 780M totalmente ativada, basta rodar este comando no terminal:

```bash
docker run -it \
  --device=/dev/kfd \
  --device=/dev/dri \
  --security-opt seccomp=unconfined \
  --group-add video \
  --ipc=host \
  --shm-size 8G \
  -e HSA_OVERRIDE_GFX_VERSION=11.0.0 \
  rocm/pytorch:latest
```

### O que este comando mágico faz?
* `--device=/dev/kfd` e `--device=/dev/dri`: Dão ao container acesso direto aos componentes físicos da sua placa de vídeo AMD.
* `-e HSA_OVERRIDE_GFX_VERSION=11.0.0`: É o **truque de ouro**. Ele diz ao container para contornar a falta de suporte oficial para a Radeon 780M (gfx1103) e fazê-la rodar se passando por uma placa de desktop compatível (gfx1100).
* `rocm/pytorch:latest`: Baixa automaticamente a versão mais recente e estável do ambiente oficial de inteligência artificial da AMD.

*(A primeira execução pode demorar alguns minutos para baixar os arquivos. Assim que terminar, você será colocado diretamente dentro do ambiente da AMD).*

---

## 🧪 Passo 3: Testar e Validar

Uma vez que o comando acima terminou e você está dentro do container, vamos validar se a sua placa está sendo reconhecida.

1. Digite no terminal para abrir o interpretador de códigos:
   ```bash
   python3
   ```
2. Cole as duas linhas abaixo, uma de cada vez, apertando **Enter** após cada uma:
   ```python
   import torch
   print(torch.cuda.is_available())
   ```
3. Se o terminal retornar **`True`**, sua GPU Radeon 780M está oficialmente integrada e pronta para processar IA com aceleração por hardware!
4. Para sair do ambiente de testes, digite:
   ```python
   exit()
   ```
5. Para fechar o container do Docker e voltar ao seu terminal normal, digite:
   ```bash
   exit
   ```

---

## 🤖 Bônus: Rodar o Ollama (Estilo ChatGPT Offline) com 1 Comando

Se o seu objetivo principal é rodar modelos de linguagem locais (como Llama 3 ou Gemma) em uma interface de chat de forma totalmente offline, você pode subir o Ollama oficial para placas AMD em segundo plano com este comando:

```bash
docker run -d \
  --device=/dev/kfd \
  --device=/dev/dri \
  -e HSA_OVERRIDE_GFX_VERSION=11.0.0 \
  -v ollama:/root/.ollama \
  -p 11434:11434 \
  --name ollama \
  ollama/ollama:rocm
```

Uma vez rodando, você pode usar qualquer interface visual amigável (como o Open WebUI ou o app de desktop Chatbox) apontando para o seu endereço local para ter o seu próprio assistente de IA ultra-rápido rodando na Radeon 780M do seu Geekom A7 Max.

---

### ⚠️ O Comportamento de "Warm-up" (Aquecimento)
É normal que, na primeira vez que você executar uma tarefa pesada de inteligência artificial (como processar um texto muito longo ou abrir um modelo grande), a tela do seu computador dê uma piscada rápida de um segundo. Isso é o driver de vídeo integrado (`amdgpu`) se ajustando e reiniciando com a nova carga de processamento de IA. Nas execuções seguintes, o funcionamento será imediato, fluido e sem interrupções.
