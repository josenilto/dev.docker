# Docker Monitoring & Logging Stack

## Visão Geral

Este projeto contém uma stack completa de **Monitoramento**, **Visualização** e **Gerenciamento de Logs** usando Docker e Docker Compose.

## Serviços Inclusos

### 📊 Stack de Monitoramento

| Serviço | Porta | Função |
|---------|-------|--------|
| **Prometheus** | 9090 | Coleta e armazenamento de métricas |
| **InfluxDB** | 8086 | Banco de dados de séries temporais |
| **Grafana** | 3000 | Visualização e dashboards |
| **Node Exporter** | 9100 | Coleta de métricas do sistema |

### 📋 Stack de Logging

| Serviço | Porta | Função |
|---------|-------|--------|
| **Graylog** | 9000 | Gerenciamento centralizado de logs |
| **Elasticsearch** | 9200 | Engine de busca e armazenamento |
| **MongoDB** | 27017 | Banco de dados de configurações |

## Iniciando a Stack

### Pré-requisitos
- Docker Engine 20.10+
- Docker Compose 2.0+
- 4GB RAM mínimo
- 10GB espaço em disco

### Executar tudo

```bash
# Clonar/navegar para o diretório
cd dev.docker-main

# Construir as imagens
docker-compose build

# Iniciar os containers
docker-compose up -d

# Verificar status
docker-compose ps
```

### Acessar os serviços

| Serviço | URL | Credenciais |
|---------|-----|-------------|
| **Grafana** | http://localhost:3000 | admin / admin |
| **Prometheus** | http://localhost:9090 | - |
| **InfluxDB** | http://localhost:8086 | admin / admin |
| **Graylog** | http://localhost:9000 | admin / admin |
| **Elasticsearch** | http://localhost:9200 | - |

## Fluxo de Dados

```
┌─────────────────────────────────────────────────────────────┐
│                    NODE EXPORTER                            │
│              (Coleta de Métricas do Sistema)                │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                     PROMETHEUS                              │
│              (Armazenamento de Métricas)                    │
└────────────────────────┬────────────────────────────────────┘
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
    ┌───────────┐  ┌──────────┐  ┌──────────────┐
    │ GRAFANA   │  │ INFLUXDB │  │ ELASTICSEARCH│
    │(Dashboards)  │(Séries)  │  │(Logs/Search) │
    └───────────┘  └──────────┘  └──────────────┘
        │                                │
        └────────────────┬────────────────┘
                         ▼
                    ┌──────────┐
                    │ GRAYLOG  │
                    │(Logs Centralizados)
                    └──────────┘
```

## Configurações Automáticas

✅ **Grafana**
- Dashboard pré-configurado (System Monitoring)
- Datasources automáticos:
  - Prometheus (padrão)
  - InfluxDB
  - Elasticsearch

✅ **Prometheus**
- Targets pré-configurados:
  - Prometheus (auto-monitoramento)
  - Node Exporter
  - Grafana
  - InfluxDB
  - Docker
- Alertas predefinidos (CPU, Memória, Disco, Targets)

✅ **Graylog**
- MongoDB para configurações
- Elasticsearch para indexação de logs
- Inputs disponíveis (Syslog, GELF, Raw/Plaintext)

## Estrutura do Projeto

```
dev.docker-main/
├── docker-compose.yml          # Orquestração de serviços
├── prometheus/
│   ├── Dockerfile
│   ├── config/
│   │   ├── prometheus.yml
│   │   └── alerts.yml
│   └── readme.md
├── grafana/
│   ├── Dockerfile
│   ├── dashboard/
│   │   ├── dashboard-config.json
│   │   └── readme.md
│   ├── datasource/
│   │   ├── datasource.yaml
│   │   └── readme.md
│   ├── loki/
│   │   ├── Dockerfile
│   │   ├── config/
│   │   │   └── loki-config.yaml
│   │   └── readme.md
│   └── readme.md
├── influxdb/
│   ├── Dockerfile
│   └── readme.md
├── graylog/
│   ├── Dockerfile
│   ├── config/
│   └── readme.md
├── promtail/
│   ├── promtail-config.yaml
│   └── readme.md
└── compose/
    ├── apache2/
    ├── app java/
    └── grafana/
```

## Exemplos de Uso

### 1. Visualizar Métricas do Sistema

1. Acesse Grafana: http://localhost:3000
2. Login com admin/admin
3. Vá para **Dashboards > System Monitoring Dashboard**
4. Visualize métricas de CPU, Memória, Disco, Rede

### 2. Enviar Logs para Graylog

**Via Syslog:**
```bash
echo "<34>Oct 11 22:14:15 myhostname tag: message" | nc -w 0 -u localhost 514
```

**Via GELF:**
```bash
echo '{"version":"1.1","host":"example.com","short_message":"test","timestamp":'$(date +%s)',"level":1}' | nc -w 0 -u localhost 1514
```

**Via Raw:**
```bash
echo "Test log message" | nc -w 0 localhost 5555
```

### 3. Criar Queries PromQL no Prometheus

Acesse: http://localhost:9090/graph

Exemplos:
```promql
# CPU por 5 minutos
rate(node_cpu_seconds_total[5m]) * 100

# Memória disponível
node_memory_MemAvailable_bytes

# Disco usado
node_filesystem_used_bytes
```

### 4. Integrar InfluxDB com Grafana

1. Em Grafana, vá para **Configuration > Data Sources**
2. Graylog já está pré-configurado
3. Para modificar token, edite: `grafana/datasource/datasource.yaml`

## Parar e Remover

```bash
# Parar containers (mantém volumes)
docker-compose down

# Remover containers e volumes
docker-compose down -v

# Remover também as imagens
docker-compose down -v --rmi all
```

## Troubleshooting

### Grafana não conecta ao Prometheus
```bash
# Verificar conectividade de rede
docker exec grafana curl -v http://prometheus:9090

# Verificar logs do Grafana
docker logs grafana
```

### Graylog não inicia
```bash
# Verificar logs
docker logs graylog

# Verificar se MongoDB e Elasticsearch estão rodando
docker ps | grep -E 'mongodb|elasticsearch'
```

### Alto uso de disco
```bash
# Limpar volumes não utilizados
docker volume prune

# Ver tamanho dos volumes
docker system df
```

## Segurança em Produção

⚠️ **Importante:**

1. **Altere as senhas padrão:**
   - Grafana admin/admin
   - InfluxDB admin/admin
   - Graylog admin/admin

2. **Altere secrets:**
   - `GRAYLOG_PASSWORD_SECRET` (Graylog)
   - `GRAYLOG_ROOT_PASSWORD_SHA2` (Graylog)

3. **Configure SSL/TLS** para todos os serviços

4. **Implemente autenticação** (LDAP, OAuth2)

5. **Configure backups** periódicos

6. **Monitore recursos** (CPU, Memória, Disco)

## Referências

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [InfluxDB Documentation](https://docs.influxdata.com/)
- [Graylog Documentation](https://docs.graylog.org/)
- [Elasticsearch Documentation](https://www.elastic.co/guide/en/elasticsearch/reference/)

## Suporte

Para problemas específicos, consulte:
- `prometheus/readme.md` - Documentação do Prometheus
- `grafana/readme.md` - Documentação do Grafana
- `influxdb/readme.md` - Documentação do InfluxDB
- `graylog/readme.md` - Documentação do Graylog
