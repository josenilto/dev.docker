# 🔗 Integração Ubuntu 20.04 com Stack de Monitoramento

## 📋 Visão Geral

Guia completo para monitorar o **Ubuntu 20.04 com Apache2** usando:
- ✅ Grafana (Visualização)
- ✅ Loki (Logs)
- ✅ Zabbix (Alertas e Monitoramento Centralizado)
- ✅ Prometheus (Métricas)

---

## 🏗️ Arquitetura de Integração

```
┌──────────────────────────────────────────┐
│  Ubuntu 20.04 Container                  │
│  - Apache2 (porta 80)                    │
│  - Zabbix Agent (10050)                  │
│  - Node Exporter (9100)                  │
│  - Promtail (envia logs)                 │
└──────────────────────────────────────────┘
     ↓              ↓              ↓
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Zabbix   │  │Prometheus│  │ Promtail │
│ Server   │  │ Scraper  │  │→ Loki    │
└──────────┘  └──────────┘  └──────────┘
     ↓              ↓              ↓
┌─────────────────────────────────────────┐
│         Grafana Dashboards              │
│  - System Metrics (CPU, Memory, Disk)  │
│  - Apache2 Metrics                      │
│  - Logs (Apache access/error)          │
│  - Zabbix Triggers & History           │
└─────────────────────────────────────────┘
```

---

## 🔧 Configuração Inicial

### 1. **Iniciar Stack Completo**

```bash
# Build do Ubuntu container
docker-compose build ubuntu-20-04

# Iniciar Ubuntu com dependências
docker-compose up -d mysql zabbix-server zabbix-frontend
docker-compose up -d prometheus grafana loki
docker-compose up -d ubuntu-20-04
```

### 2. **Verificar Conectividade**

```bash
# Verificar container rodando
docker-compose ps ubuntu-20-04

# Verificar status
curl http://localhost:8081/status.php
```

---

## 📊 Integração com Zabbix

### 1. **Verificar Host no Zabbix**

```
1. Acessar Zabbix Frontend: http://localhost:8080
2. Configuration → Hosts
3. Procurar por "Ubuntu-20.04-Server"
4. Status deve ser "Available" (verde)
```

### 2. **Métricas Disponíveis no Zabbix**

```yaml
Sistema:
  - CPU Load Average
  - Memory Usage
  - Disk Usage
  - Process Count
  - Network Traffic
  - Uptime

Apache2:
  - Total Accesses
  - Requests per second
  - Bytes per second
  - Worker processes
```

### 3. **Criar Triggers em Zabbix**

**Exemplo: Alerta se CPU > 80%**

```
Configuration → Triggers → Create trigger
  Name: Ubuntu - High CPU Load
  Expression: {Ubuntu-20.04-Server:system.cpu.load.avg(5m)}>80
  Severity: Warning
```

**Exemplo: Alerta se Disco > 90%**

```
Configuration → Triggers → Create trigger
  Name: Ubuntu - Low Disk Space
  Expression: {Ubuntu-20.04-Server:vfs.fs.used[/].last()}/{Ubuntu-20.04-Server:vfs.fs.total[/].last()}*100>90
  Severity: Critical
```

---

## 📈 Integração com Grafana

### 1. **Adicionar Datasource Zabbix**

```
Grafana → Configuration → Data Sources → Add data source
Type: Zabbix
Name: Zabbix Ubuntu
URL: http://zabbix-server:10051
Zabbix API URL: http://zabbix-server/api_jsonrpc.php
Username: Admin
Password: zabbix
```

### 2. **Criar Dashboard: Ubuntu System Overview**

**Painel 1: CPU Usage**
```
Datasource: Zabbix
Host: Ubuntu-20.04-Server
Item: CPU load average (5m)
Type: Graph
```

**Painel 2: Memory Usage**
```
Datasource: Zabbix
Host: Ubuntu-20.04-Server
Item: Memory used
Type: Gauge
```

**Painel 3: Disk Usage**
```
Datasource: Zabbix
Host: Ubuntu-20.04-Server
Item: Disk used on root
Type: Piechart
```

**Painel 4: Apache2 Status**
```
Datasource: Zabbix
Host: Ubuntu-20.04-Server
Item: Apache requests per second
Type: Graph
```

### 3. **Criar Dashboard: Apache2 Monitoring**

**Painel 1: Total Accesses**
```
Datasource: Prometheus
Query: apache2_total_accesses{instance="ubuntu-20-04:9100"}
```

**Painel 2: Requests/sec**
```
Datasource: Prometheus
Query: apache2_requests_per_sec{instance="ubuntu-20-04:9100"}
```

**Painel 3: Worker Processes**
```
Datasource: Prometheus
Query: apache2_busy_workers{instance="ubuntu-20-04:9100"}
Query: apache2_idle_workers{instance="ubuntu-20-04:9100"}
```

---

## 📝 Integração com Loki (Logs)

### 1. **Verificar Logs no Grafana**

```
Grafana → Explore → Datasource: Loki
```

### 2. **Filtros Disponíveis**

**Apache2 Access Logs:**
```
{job="apache2-access", host="ubuntu-20-04"}
```

**Apache2 Error Logs:**
```
{job="apache2-error", host="ubuntu-20-04"}
```

**System Logs:**
```
{job="system", host="ubuntu-20-04"}
```

**Zabbix Agent Logs:**
```
{job="zabbix-agent", host="ubuntu-20-04"}
```

### 3. **Criar Dashboard: Ubuntu Logs**

**Painel 1: Apache Access Logs**
```
Datasource: Loki
Query: {job="apache2-access", host="ubuntu-20-04"}
Line limit: 100
```

**Painel 2: Apache Error Logs**
```
Datasource: Loki
Query: {job="apache2-error", host="ubuntu-20-04"} | json
```

**Painel 3: System Events**
```
Datasource: Loki
Query: {job="system", host="ubuntu-20-04"} | level="ERROR"
```

---

## 📊 Exemplos de Queries

### Prometheus Queries

**CPU Usage:**
```promql
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

**Memory Usage:**
```promql
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
```

**Disk Usage:**
```promql
(1 - (node_filesystem_avail_bytes / node_filesystem_size_bytes)) * 100
```

**Apache Requests:**
```promql
rate(apache2_total_accesses[5m])
```

### Loki Queries (LogQL)

**Apache Errors (últimas 24h):**
```
{job="apache2-error", host="ubuntu-20-04"}
| timerange(24h)
```

**Status HTTP 5xx:**
```
{job="apache2-access", host="ubuntu-20-04"}
| json status_code="status_code"
| status_code >= "500"
```

**Performance por endpoint:**
```
{job="apache2-access", host="ubuntu-20-04"}
| json method="method", path="path"
| stats avg(duration_ms) by path
```

---

## 🔔 Alertas Integrados

### Exemplo 1: Alerta de Espaço em Disco

```yaml
# Zabbix
Trigger: Low Disk Space
Expression: {Ubuntu-20.04-Server:vfs.fs.used[/]}>90
Action: Send email to admin@example.com
```

### Exemplo 2: Alerta de Apache Down

```yaml
# Zabbix
Trigger: Apache Process Down
Expression: {Ubuntu-20.04-Server:proc.num[apache2]}=0
Severity: Critical
Action: Restart Apache + Email
```

### Exemplo 3: Taxa de Erro HTTP Alta

```yaml
# Grafana Alert
Datasource: Prometheus
Query: rate(apache2_errors[5m]) > 10
For: 5m
Send to: Slack channel
```

---

## 🛠️ Troubleshooting

### Problema: Ubuntu não aparece em Zabbix

**Solução:**
```bash
# Verificar se Zabbix Agent está rodando
docker-compose exec ubuntu-20-04 systemctl status zabbix-agent

# Verificar logs
docker-compose exec ubuntu-20-04 tail -f /var/log/zabbix/zabbix_agentd.log

# Testar conexão
docker-compose exec ubuntu-20-04 telnet zabbix-server 10051
```

### Problema: Prometheus não consegue scrape

**Solução:**
```bash
# Verificar Node Exporter
docker-compose exec ubuntu-20-04 pgrep -f node_exporter

# Testar conexão
curl http://localhost:9101/metrics
```

### Problema: Loki não recebe logs

**Solução:**
```bash
# Verificar Promtail
docker-compose exec ubuntu-20-04 pgrep -f promtail

# Ver logs do Promtail
docker-compose logs -f ubuntu-20-04 | grep promtail

# Testar push para Loki
curl -X POST -H "Content-Type: application/json" \
  -d '{"streams":[{"stream":{"test":"test"},"values":[["'$(date +%s%N)'","test"]]}]}' \
  http://localhost:3100/loki/api/v1/push
```

### Problema: Apache2 não inicia

**Solução:**
```bash
# Testar configuração
docker-compose exec ubuntu-20-04 apache2ctl configtest

# Ver logs de erro
docker-compose exec ubuntu-20-04 tail -f /var/log/apache2/error.log

# Reiniciar
docker-compose restart ubuntu-20-04
```

---

## 📚 Dashboard Recommendations

### Nível 1: Overview (para gerentes)
- Saúde geral do sistema
- Alertas críticos
- Uptime

### Nível 2: System Admin
- CPU/Memory/Disk detalhado
- Network traffic
- Process list
- Trigger history

### Nível 3: Application
- Apache2 requests/errors
- Response times
- User agents
- Top 10 URLs

### Nível 4: DevOps
- Log streams
- Correlação de eventos
- Métricas customizadas
- Histórico de performance

---

## ✅ Checklist de Configuração

- [ ] Ubuntu container rodando
- [ ] Zabbix Agent conectado e disponível
- [ ] Node Exporter respondendo em 9100
- [ ] Promtail enviando logs para Loki
- [ ] Apache2 acessível em 8081
- [ ] Host aparece em Zabbix
- [ ] Métricas em Prometheus
- [ ] Dashboard criado em Grafana
- [ ] Triggers configurados
- [ ] Alertas testados

---

## 📞 Suporte

Para problemas específicos:

1. Verificar container status: `docker-compose ps`
2. Ver logs: `docker-compose logs ubuntu-20-04`
3. Executar health check: `curl http://localhost:8081/status.php`
4. Coletar métricas: `docker-compose exec ubuntu-20-04 /usr/local/bin/collect-metrics.sh`

---

## 📚 Referências

- [Zabbix Documentation](https://www.zabbix.com/documentation/)
- [Grafana Dashboards](https://grafana.com/grafana/dashboards/)
- [Prometheus Queries](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [LogQL Documentation](https://grafana.com/docs/loki/latest/logql/)
