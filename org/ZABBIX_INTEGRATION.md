# 🔗 Integração Zabbix com Stack de Monitoramento

## 📋 Visão Geral

Este documento descreve como o Zabbix Server está integrado com o stack completo de monitoramento (Prometheus, Grafana, InfluxDB, Loki).

---

## 🏗️ Arquitetura Completa

```
┌────────────────────────────────────────────────────────────┐
│                    AGENTES MONITORADOS                     │
│  (Servidores, VMs, Containers com Zabbix Agents)          │
└────────────────────────────────────────────────────────────┘
         ↓ (porta 10051)
┌────────────────────────────────────────────────────────────┐
│                   ZABBIX SERVER                            │
│  - Recebe dados dos agentes                               │
│  - Processa e valida                                      │
│  - Aciona alertas                                         │
│  - Armazena em MySQL                                      │
└────────────────────────────────────────────────────────────┘
    ↓           ↓              ↓              ↓
┌────────┐ ┌────────┐   ┌──────────┐   ┌──────────┐
│ MySQL  │ │Grafana │   │Prometheus│   │  Loki    │
│(DB)    │ │(UI)    │   │(Métricas)│   │(Logs)    │
└────────┘ └────────┘   └──────────┘   └──────────┘
```

---

## 📊 Componentes Zabbix

### 1. **Zabbix Server** (zabbix-server)

**Responsabilidades:**
- ✅ Coleta dados de agentes na porta 10051
- ✅ Processa triggers (regras)
- ✅ Aciona alertas
- ✅ Armazena dados em MySQL
- ✅ Executa scripts customizados

**Configuração:**
```yaml
- Container: zabbix-server
- Porta: 10051 (TCP)
- Banco: MySQL (localhost:3306)
- Volume: /usr/lib/zabbix/
```

### 2. **Zabbix Frontend** (zabbix-frontend)

**Responsabilidades:**
- ✅ Interface web para Zabbix
- ✅ Dashboards
- ✅ Configuração de hosts
- ✅ Visualização de alertas

**Acesso:**
```
URL: http://localhost:8080
Usuário: Admin
Senha: zabbix
```

### 3. **MySQL** (mysql)

**Responsabilidades:**
- ✅ Armazenar configurações do Zabbix
- ✅ Histórico de valores
- ✅ Alertas
- ✅ Usuários e permissões

**Credenciais:**
```
Host: mysql:3306
Database: zabbix
Usuário: zabbix
Senha: zabbix_password
```

---

## 🔄 Integração com Grafana

### 1. **Grafana como Frontend Alternativo**

Você pode usar Grafana para visualizar dados do Zabbix:

**Passo 1: Adicionar Datasource**
```
Grafana → Configuration → Data Sources → Add Data Source
```

**Passo 2: Configurar Zabbix**
```
Name: Zabbix
Type: Zabbix
URL: http://zabbix-server:10051
```

**Passo 3: Configurar Autenticação**
```
Zabbix API URL: http://zabbix-server/api_jsonrpc.php
Usuário: Admin
Senha: zabbix
```

### 2. **Dashboards em Grafana**

**Exemplo de Dashboard:**
```json
{
  "title": "Zabbix System Monitoring",
  "panels": [
    {
      "title": "CPU Usage",
      "targets": [
        {
          "datasource": "Zabbix",
          "host": "Linux servers",
          "item": "CPU load"
        }
      ]
    },
    {
      "title": "Memory Usage",
      "targets": [
        {
          "datasource": "Zabbix",
          "host": "Linux servers",
          "item": "Memory used"
        }
      ]
    }
  ]
}
```

---

## 📈 Integração com Prometheus

### Exportador Zabbix para Prometheus

**Como configurar:**

```yaml
# Em docker-compose.yml (adicionar)
zabbix-prometheus-exporter:
  image: zabbix/zabbix-exporter-prom-exporter:latest
  container_name: zabbix-exporter
  ports:
    - "9713:9713"
  environment:
    - ZABBIX_SERVER_HOST=zabbix-server
    - ZABBIX_SERVER_PORT=10051
  networks:
    - monitoring-network
  depends_on:
    - zabbix-server
```

**Adicionar ao Prometheus:**

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'zabbix'
    static_configs:
      - targets: ['zabbix-exporter:9713']
```

---

## 📝 Integração com Loki (Logs)

### Enviar Logs do Zabbix para Loki

**Opção 1: Via Promtail**

```yaml
# promtail-config.yaml (adicionar job)
scrape_configs:
  - job_name: zabbix
    static_configs:
      - targets:
          - localhost
        labels:
          job: zabbix
          __path__: /var/log/zabbix/*.log
```

**Opção 2: Via Webhook Zabbix**

```
Zabbix Server → Alertas → Media Script → Loki HTTP API
```

---

## 🎯 Casos de Uso

### 1. **Monitoramento de Infraestrutura**

```
Zabbix Agent em cada servidor
    ↓
Coleta CPU, Memória, Disco, Rede
    ↓
Zabbix Server processa
    ↓
Alertas disparados se > limiar
    ↓
Notificação via Email/Slack/SMS
```

### 2. **Monitoramento de Aplicações**

```
Aplicação expõe métricas (HTTP/SNMP)
    ↓
Zabbix Agent coleta via custom script
    ↓
Zabbix Server valida
    ↓
Grafana visualiza em tempo real
```

### 3. **Correlação com Logs**

```
Alerta Zabbix disparado
    ↓
Log registrado em Graylog/Loki
    ↓
Análise cruzada de causa raiz
    ↓
Ação automática ou manual
```

---

## 🚀 Como Começar

### 1. **Iniciar Stack Zabbix**

```bash
docker-compose up -d mysql zabbix-server zabbix-frontend
```

### 2. **Acessar Frontend**

```
URL: http://localhost:8080
Usuário: Admin
Senha: zabbix
```

### 3. **Adicionar Hosts para Monitoramento**

```
Frontend → Configuration → Hosts → Create host
```

**Exemplo de Host:**

```
Hostname: Linux Server 1
Visible name: Linux Server 1
IP Address: 192.168.1.100
Port: 10050
Agent interface: Enabled
```

### 4. **Instalar Zabbix Agent nos Servidores**

**Linux (Ubuntu/Debian):**
```bash
wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-4+ubuntu22.04_all.deb
dpkg -i zabbix-release_6.0-4+ubuntu22.04_all.deb
apt-get update
apt-get install zabbix-agent
```

**Configurar Agent:**
```bash
# /etc/zabbix/zabbix_agentd.conf
Server=<ZABBIX_SERVER_IP>
ServerActive=<ZABBIX_SERVER_IP>:10051
Hostname=Linux Server 1
```

**Iniciar Agent:**
```bash
systemctl start zabbix-agent
systemctl enable zabbix-agent
```

### 5. **Verificar Conexão**

```bash
docker-compose logs -f zabbix-server | grep "accepting connections"
```

---

## 🔍 Troubleshooting

### Problema: Agent não conecta ao Server

**Verificar:**
```bash
# No servidor do agent
telnet <ZABBIX_SERVER_IP> 10051

# No container Zabbix
docker-compose logs zabbix-server
docker-compose exec zabbix-server netstat -tuln | grep 10051
```

**Solução:**
- Verificar firewall
- Verificar hostname/IP no config
- Reiniciar agent

### Problema: Frontend retorna erro

**Verificar:**
```bash
docker-compose logs zabbix-frontend
docker-compose exec zabbix-frontend curl http://zabbix-server:10051
```

### Problema: MySQL não conecta

**Verificar:**
```bash
docker-compose exec mysql mysql -uroot -p
```

**Solução:**
- Aguardar inicialização do MySQL (40s)
- Verificar variáveis de ambiente

---

## 📊 Métricas Principais

### Padrão Linux

| Métrica | Descrição | Unidade |
|---------|-----------|---------|
| system.cpu.load | Carga do CPU | % |
| system.memory.used | Memória usada | bytes |
| vfs.fs.used | Disco usada | bytes |
| net.if.out | Tráfego de saída | bytes |
| net.if.in | Tráfego de entrada | bytes |
| system.uptime | Tempo ligado | segundos |

---

## 🔐 Segurança

### Boas Práticas

✅ **Senhas:**
- Mudar padrão "zabbix" em produção
- Usar hash SHA2 para senhas

✅ **Network:**
- Isolar em rede interna
- Usar TLS para agentes remotos

✅ **Firewall:**
- Abrir 10051 apenas para agentes conhecidos
- Frontend atrás de proxy/VPN

---

## 📚 Referências

- [Documentação Zabbix 6.0](https://www.zabbix.com/documentation/6.0/)
- [Integração Grafana + Zabbix](https://grafana.com/docs/grafana/latest/datasources/zabbix/)
- [API Zabbix](https://www.zabbix.com/documentation/current/manual/api)

---

## 📞 Suporte

Para problemas específicos:

1. Verificar logs: `docker-compose logs [service]`
2. Acessar Frontend: `http://localhost:8080`
3. Verificar status dos hosts
4. Consultar documentação oficial do Zabbix
