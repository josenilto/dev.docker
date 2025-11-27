# 📚 Documentação Completa --- Wiki do Projeto

Bem-vindo à documentação completa do projeto. Este documento serve como
base para a Wiki oficial, oferecendo instruções detalhadas, arquitetura,
tutoriais e material de referência.

------------------------------------------------------------------------

# 📌 1. Introdução

Este projeto tem como objetivo fornecer uma stack completa baseada em
**Docker** e **Docker Compose**, permitindo o provisionamento de
ambientes escaláveis, reproducíveis e fáceis de manter.

------------------------------------------------------------------------

# 🏗 2. Arquitetura do Sistema

A arquitetura segue uma abordagem modular:

    +------------------------+
    |        Cliente         |
    +-----------+------------+
                |
                v
    +-----------+------------+
    |      NGINX / Proxy     |
    +-----------+------------+
                |
                v
    +------------------------+
    |     Aplicação Web      |
    |  (Containerizada)      |
    +------------------------+
                |
                v
    +------------------------+
    |        Database        |
    +------------------------+

### Componentes Principais

-   **Nginx Proxy** --- Gateway de entrada.
-   **Aplicação** --- Serviço principal containerizado.
-   **Banco de Dados** --- PostgreSQL / MySQL (dependendo da stack).
-   **Logs** --- Integração opcional com ELK / Grafana Loki.

------------------------------------------------------------------------

# 🧱 3. Requisitos

  Componente       Versão Requerida
  ---------------- -------------------
  Ubuntu           20.04 ou superior
  Docker           24 ou superior
  Docker Compose   v2 ou superior
  CPU              2+ cores
  RAM              4GB+
  Disco            10GB+

------------------------------------------------------------------------

# ⚙️ 4. Instalação do Docker e Compose

Procedimentos validados para Ubuntu 20.04+.

### Passo 1 --- Atualizar sistema

    sudo apt-get update && sudo apt-get upgrade -y

### Passo 2 --- Pacotes essenciais

    sudo apt-get install -y software-properties-common apt-transport-https ca-certificates curl gnupg lsb-release

### Passo 3 --- Repositório oficial Docker

    sudo mkdir -m 0755 -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

### Passo 4 --- Adicionar repositório Docker

    echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu   $(. /etc/os-release && echo "$VERSION_CODENAME") stable" |   sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

### Passo 5 --- Instalar Docker Engine

    sudo apt-get update
    sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

------------------------------------------------------------------------

# 🔧 5. Configuração Pós-instalação

### Adicionar usuário ao grupo docker

    sudo usermod -aG docker $USER

### Reiniciar sessão:

    newgrp docker

------------------------------------------------------------------------

# ▶️ 6. Execução do Ambiente

### Subir serviços

    docker compose up -d

### Parar serviços

    docker compose down

------------------------------------------------------------------------

# 🛠 7. Comandos Úteis

  Ação                         Comando
  ---------------------------- -------------------------------
  Ver containers               `docker ps -a`
  Entrar no container          `docker exec -it <nome> bash`
  Ver logs                     `docker logs -f <nome>`
  Remover containers parados   `docker container prune`

------------------------------------------------------------------------

# 📂 8. Estrutura do Projeto

    /project
    ├─ docker/
    │  ├─ nginx/
    │  ├─ db/
    ├─ src/
    │  ├─ app/
    │  └─ api/
    ├─ scripts/
    ├─ docs/
    └─ docker-compose.yml

------------------------------------------------------------------------

# 🧩 9. Troubleshooting (Problemas Comuns)

### Docker sem permissão

**Erro:**

    permission denied

**Solução:**\
Execute:

    sudo usermod -aG docker $USER

### Porta já usada

    Error: port is already allocated

**Solução:**

    sudo lsof -i :PORT
    sudo kill -9 PID

------------------------------------------------------------------------

# 🧱 10. Boas Práticas

-   Fixar versões no `docker-compose.yml`
-   Utilizar `.env` para variáveis sensíveis
-   Evitar imagens `latest`
-   Usar volumes nomeados
-   Criar healthchecks

------------------------------------------------------------------------

# 🤝 11. Contribuição

Contribuições são bem-vindas!\
Crie uma **issue** ou envie um **Pull Request**.

------------------------------------------------------------------------

# 📜 12. Licença

Distribuído sob a licença **MIT**.

------------------------------------------------------------------------

# 🏁 Fim

Obrigado por utilizar esta documentação.
