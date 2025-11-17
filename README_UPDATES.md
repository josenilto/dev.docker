# Docker Monitoring & Logging Stack - Estrutura Atualizada

## Visão Geral

Este projeto agora conta com **Loki integrado ao projeto Grafana**, fornecendo uma solução completa de monitoring e logging.

## 📁 Nova Estrutura

```
dev.docker-main/
├── docker-compose.yml
├── STACK.md                    # Documentação principal
├── LOKI_INTEGRATION.md        # Guia de integração Loki
├── README_UPDATES.md          # Este arquivo
│
├── compose/
│   ├── prometheus/            # Stack de monitoramento
│   │   ├── Dockerfile
│   │   ├── config/
│   │   │   ├── prometheus.yml
│   │   │   └── alerts.yml
│   │   └── readme.md
│   │
│   ├── grafana/               # Stack Grafana + Loki
│   │   ├── Dockerfile
│   │   ├── dashboard/
│   │   │   ├── dashboard-config.json
│   │   │   └── readme.md
│   │   ├── datasource/
│   │   │   ├── datasource.yaml
│   │   │   └── readme.md
│   │   ├── loki/             # ✨ NOVO: Loki dentro de Grafana
│   │   │   ├── Dockerfile
│   │   │   ├── config/
│   │   │   │   └── loki-config.yaml
│   │   │   └── readme.md
│   │   └── readme.md
│   │
│   ├── influxdb/              # Banco de dados time-series
│   │   ├── Dockerfile
│   │   └── readme.md
│   │
│   ├── graylog/               # Stack de logging centralizado
│   │   ├── Dockerfile
│   │   ├── config/
│   │   └── readme.md
│   │
│   └── promtail/              # ✨ NOVO: Coleta de logs para Loki
│       ├── promtail-config.yaml
│       └── readme.md
│
├── compose/apache2/           # Outros serviços
├── compose/app java/
└── ... (outros diretórios)
```

## 🔄 Mudanças Realizadas

### 1. **Loki Movido para Grafana**
- **Antes:** `compose/loki/`
- **Depois:** `compose/grafana/loki/`
- **Razão:** Melhor organização, já que Loki é parte do stack de visualização

### 2. **docker-compose.yml Atualizado**
```yaml
# Build path atualizado para Loki
loki:
  build: ./compose/grafana/loki    # ← Novo caminho
  container_name: loki
  ports:
    - "3100:3100"
```

### 3. **Datasources do Grafana**
Agora inclui automaticamente **Loki** como datasource:
```yaml
- name: Loki
  type: loki
  url: http://loki:3100
  isDefault: false
```

## 🚀 Como Usar

### Build e Start
```bash
cd dev.docker-main

# Construir imagens
docker-compose build

# Iniciar stack completo
docker-compose up -d

# Verificar status
docker-compose ps
```

### Acessar Serviços
| Serviço | URL | Usuário/Senha |
|---------|-----|---------------|
| **Grafana** | http://localhost:3000 | admin/admin |
| **Prometheus** | http://localhost:9090 | - |
| **Loki** | http://localhost:3100 | - |
| **InfluxDB** | http://localhost:8086 | admin/admin |
| **Graylog** | http://localhost:9000 | admin/admin |

## 📊 Fluxo de Dados Completo

```
┌─────────────────────────────────────────────┐
│          SISTEMA / APLICAÇÕES               │
└─────────────┬───────────────────────────────┘
              │
    ┌─────────┴──────────┐
    │                    │
    ▼                    ▼
  MÉTRICAS           LOGS
    │                    │
    ├─> Node Exporter   │
    │   └─> Prometheus   │ ├─> Promtail ──┐
    │       └─> Grafana  │ │               │
    │                    └─> Graylog      │
    │                    │   └─> ES/Mongo │
    │                    │                 │
    └────────────────────┼─────────────────┤
                         │                 │
                         └─> Loki <────────┘
                             │
                             ▼
                         GRAFANA
                    (Visualização Unificada)
```

## 🔧 Configurações Importantes

### Dockerfile do Grafana (Atualizado)
```dockerfile
# Plugins instalados
ENV GF_INSTALL_PLUGINS=grafana-piechart-panel,grafana-elasticsearch-datasource-plugin

# Datasources provisionados automaticamente
# - Prometheus
# - InfluxDB
# - Elasticsearch
# - Loki ✨ (novo)
```

### Loki Dentro de Grafana
- **Porta:** 3100
- **Armazenamento:** BoltDB + Filesystem
- **Volume:** `loki-chunks:/loki/chunks`
- **Health Check:** Automático a cada 30s

### Promtail
- **Configuração:** `compose/promtail/promtail-config.yaml`
- **Jobs:** System, Application, Docker
- **Destino:** Envia para Loki em http://loki:3100

## 📚 Documentação

### Documentos Principais
- **STACK.md** - Guia completo da stack de monitoramento
- **LOKI_INTEGRATION.md** - Integração Grafana + Loki
- **README_UPDATES.md** - Este arquivo (novidades)

### Documentação por Componente
- `compose/prometheus/readme.md` - Prometheus
- `compose/grafana/readme.md` - Grafana
- `compose/grafana/loki/readme.md` - Loki ✨
- `compose/grafana/dashboard/readme.md` - Dashboards
- `compose/grafana/datasource/readme.md` - Datasources
- `compose/influxdb/readme.md` - InfluxDB
- `compose/graylog/readme.md` - Graylog
- `compose/promtail/readme.md` - Promtail

## 🎯 Exemplos de Uso

### Visualizar Logs no Grafana
1. Acesse http://localhost:3000
2. Vá para **Explore**
3. Selecione datasource **Loki**
4. Query: `{job="system"}` ou `{job="application"}`

### LogQL Queries
```logql
# Ver todos os logs do sistema
{job="system"}

# Logs com erro
{job="application"} |= "error"

# Taxa de logs
rate({job="app"}[5m])

# Filtro regex
{job="app"} |~ "error.*database"
```

### Prometheus Queries
```promql
# CPU por 5 minutos
rate(node_cpu_seconds_total[5m]) * 100

# Memória
node_memory_MemAvailable_bytes

# Disco
node_filesystem_used_bytes
```

## ⚙️ Customização

### Alterar Configuração do Loki
```bash
# Editar configuração
nano compose/grafana/loki/config/loki-config.yaml

# Reconstruir
docker-compose build loki

# Reiniciar
docker-compose up -d loki
```

### Adicionar Novo Job ao Promtail
```bash
# Editar configuração
nano compose/promtail/promtail-config.yaml

# Adicionar novo scrape_config com um job_name

# Reiniciar Promtail
docker-compose restart promtail
```

## 🐛 Troubleshooting

### Loki não conecta ao Grafana
```bash
# Verificar se Loki está rodando
docker ps | grep loki

# Verificar logs
docker logs loki

# Testar conectividade
curl http://localhost:3100/ready
```

### Promtail não coleta logs
```bash
# Verificar logs do Promtail
docker logs promtail

# Verificar arquivo de posições
docker exec promtail cat /tmp/positions.yaml

# Verificar permissões de /var/log
ls -la /var/log/
```

### Alto uso de disco
```bash
# Limpar volumes não usados
docker system prune -v

# Verificar tamanho
docker system df

# Ver tamanho de volume específico
du -sh ~/.local/share/docker/volumes/loki-chunks/
```

## 📋 Checklist de Verificação

- [x] Loki movido para `compose/grafana/loki/`
- [x] docker-compose.yml atualizado com novo caminho
- [x] Dockerfile do Grafana provisionando Loki automaticamente
- [x] Datasources configurados (Prometheus, InfluxDB, Elasticsearch, Loki)
- [x] Promtail colhendo logs para Loki
- [x] Documentação atualizada
- [x] Toda estrutura testada e funcionando

## 🔐 Segurança em Produção

⚠️ **Lembrete:**
- Altere senhas padrão
- Ative autenticação em todos os serviços
- Configure SSL/TLS
- Implemente backup
- Configure firewall
- Monitore recursos

## 📞 Referências

- [Grafana Docs](https://grafana.com/docs/)
- [Loki Docs](https://grafana.com/docs/loki/latest/)
- [Prometheus Docs](https://prometheus.io/docs/)
- [InfluxDB Docs](https://docs.influxdata.com/)

---

**Status:** ✅ Estrutura reorganizada e testada
**Data:** Novembro 17, 2025
