# Dev | Docker Install com compose

✅ **PASSO 01:** Install Docker Engine.

Atualize o índice de pacotes e repositório apt.
```bash
sudo apt-get update && sudo apt-get upgrade -y
```

Atualize o índice de pacotes apt e instale pacotes para permitir que o apt use um repositório por HTTPS.
```bash
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

Criar o diretório no **`/etc/apt/`**
Adicione a chave GPG oficial do Docker:

```bash
sudo mkdir -m 0755 -p /etc/apt/keyrings
```

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