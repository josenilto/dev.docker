# Dev | Docker Install com Compose no Ubuntu 20.04

**Docker** é um de gerenciador de processos de aplicação em containers.           
Os containers são executados isolados podendo executar varias aplicações.           

São semelhantes a máquinas virtuais.            
Containers são mais portáveis, mais fáceis de usar e mais dependentes do que sistema operacionais.

Antes de inícia a instalação do **Docker** vamos verificar a versão do sistema operacional:
```bash
cat /etc/os-release
```
**Exemplo:**

NAME="Ubuntu"           
VERSION="20.04.6 LTS (Focal Fossa)"         
ID=ubuntu               
ID_LIKE=debian              
PRETTY_NAME="Ubuntu 20.04.6 LTS"            
VERSION_ID="20.04"              
HOME_URL="https://www.ubuntu.com/"          
SUPPORT_URL="https://help.ubuntu.com/"          
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"             
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"         
VERSION_CODENAME=focal          
UBUNTU_CODENAME=focal               

✅ **PASSO 01:** Vamos desativar a swap. E validar se a mesma está desabilitada.

Desativa swap:
```bash
sudo swapoff -a
```

Verificar se está desativado:
```bash
cat /etc/fstab
```
**Exemplo:**

LABEL=cloudimg-rootfs   /        ext4   defaults        0 1         
LABEL=UEFI      /boot/efi       vfat    umask=0077      0 1         

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


https://resttesttest.com/?fbclid=IwAR3smGmP1ifLZgFx86yvIYBrpwBQFbkflwofQUwjzCSYfmIPfCSmX7BwaVI
