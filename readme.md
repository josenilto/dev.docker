# Dev | Docker Install com Compose no Ubuntu 20.04

**Docker** é um de gerenciamento de processos de aplicação em containers.           
Os containers são executados isolados podendo executar varias aplicações.           
São semelhantes a máquinas virtuais.            
Containers são mais portáveis, mais fáceis de usar e mais dependentes do que sistema operacionais.

✅ **PASSO 01:**

sudo swapoff -a

✅ **PASSO 02:** Install Docker Engine.

Atualize seu Sistema, atualizado para você ter mais segurança e confiabilidade:
```bash
sudo apt-get update && sudo apt-get upgrade -y
```

Instale Pacotes de Pré-requisitos:                 
Você deve instalar alguns dos pacotes necessários.

**software-properties-common** – adiciona scripts para gerenciar o software.            
**apt-transport-https** – permite que o gerenciador de pacotes transfira os tiles e os dados através de https.      
**ca-certificates** – permite que o navegador da web e o sistema verifiquem certificados de segurança.          
**curl** – transfere dados.    

```bash
sudo apt-get install -y \
     software-properties-common \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg \
     lsb-release
```      

Adicione o Repositório no **`/etc/apt/`**
```bash
sudo mkdir -m 0755 -p /etc/apt/keyrings
```

Adiciona uma chave GPG oficial do Docker, inserindo o comando:
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

Use o seguinte comando para configurar o repositório:
```bash
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Atualize o índice de pacotes e repositório apt.
```bash
sudo apt-get update && sudo apt-get upgrade -y
```

#### ➡️ Links de referência

[<img title="Docker" align="left" alt="josenilto | Twitter" width="28px" src="https://cdn.jsdelivr.net/npm/simple-icons@v3/icons/docker.svg" />][docker]

[docker]: https://docs.docker.com/engine/install/