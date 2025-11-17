# Integração Grafana com Loki

## Visão Geral

A integração do **Grafana com Loki** fornece uma solução completa de logging que complementa o stack de monitoramento existente.

```
┌─────────────────────────────────────────────────────┐
│                  LOGS SOURCES                       │
│  (Sistema, Aplicações, Docker, etc)               │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
        ┌────────────────────────┐
        │     PROMTAIL           │
        │  (Coleta e Envia)      │
        └────────────┬───────────┘
                     │
                     ▼
        ┌────────────────────────┐
        │      LOKI              │
        │  (Armazena e Indexa)   │
        └────────────┬───────────┘
                     │
                     ▼
        ┌────────────────────────┐
        │     GRAFANA            │
        │  (Visualiza - LogQL)   │
        └────────────────────────┘
```

## Componentes

### 1. **Promtail** (Agente de Coleta)
- Coleta logs de arquivos e Docker
- Envia para Loki via gRPC/HTTP
- Configurável via YAML
- Suporta processamento em pipeline

### 2. **Loki** (Armazenamento)
- Banco de dados otimizado para logs
- Usa BoltDB + Filesystem (modo single-node)
- Escalonável (distribuído em produção)
- API LogQL para queries complexas

### 3. **Grafana** (Visualização)
- Datasource Loki integrado
- Explorador de logs avançado
- Painéis com logs em tempo real
- Alertas baseados em logs

## Estrutura do Projeto

```
compose/
├── loki/
│   ├── Dockerfile
│   ├── config/
│   │   └── loki-config.yaml
│   └── readme.md
├── promtail/
│   ├── promtail-config.yaml
│   └── readme.md
└── grafana/
    ├── Dockerfile
    ├── dashboard/
    ├── datasource/
    ├── loki/
    │   ├── Dockerfile
    │   ├── config/
    │   │   └── loki-config.yaml
    │   └── readme.md
    └── readme.md
```

## Como Funciona

### 1. Coleta de Logs (Promtail)
```yaml
Promtail lê arquivos de log especificados em scrape_configs
├── /var/log/**/*log
├── /var/lib/docker/containers/*/*-json.log
└── Outras fontes configuráveis
```

### 2. Ingestão (Loki)
```
Promtail envia logs para Loki
└── Endpoint: http://loki:3100/loki/api/v1/push
```

### 3. Armazenamento (Loki)
```
Loki comprime e indexa os logs
├── Chunks (blocos de dados)
├── Índices (labels)
└── Filesystem (/loki/chunks)
```

### 4. Visualização (Grafana)
```
Grafana consulta Loki com LogQL
├── Painel de Logs
├── Alertas
└── Dashboards
```

## Query Language - LogQL

### Exemplos Básicos

```logql
# Todos os logs com label job=app
{job="app"}

# Logs de um container específico
{container="myapp"}

# Logs contendo "error"
{job="app"} |= "error"

# Logs NÃO contendo "debug"
{job="app"} != "debug"

# Filtro regex
{job="app"} |~ "error.*database"

# Múltiplos filtros
{job="app"} |= "error" != "connection timeout"
```

### Exemplos Avançados

```logql
# Taxa de logs (logs por segundo)
rate({job="app"}[5m])

# Bytes processados
bytes_over_time({job="app"}[1h])

# Agregação por label
sum(count_over_time({job="app"}[1h])) by (instance)

# Parser de JSON
{job="app"} | json | status >= 400
```

## Iniciando a Stack com Loki

```bash
# Construir imagens
docker-compose build

# Iniciar containers
docker-compose up -d

# Verificar status
docker-compose ps
```

## Acessar os Serviços

| Serviço | URL | Função |
|---------|-----|--------|
| Loki | http://localhost:3100 | API de logs |
| Loki Metrics | http://localhost:3100/metrics | Métricas do Loki |
| Grafana | http://localhost:3000 | Visualização |

## Usando Loki no Grafana

### 1. Verificar Datasource
1. Acesse Grafana: http://localhost:3000
2. Vá para **Configuration > Data Sources**
3. Busque por "Loki"
4. Deve estar em verde (conectado)

### 2. Explorar Logs
1. Em Grafana, vá para **Explore**
2. Selecione datasource "Loki"
3. Adicione label filter: `{job="system"}`
4. Clique em "Run query"

### 3. Criar Painel com Logs
1. Novo Dashboard
2. Novo Painel
3. Datasource: Loki
4. Query LogQL: `{job="app"} |= "error"`
5. Visualização: Logs

### 4. Criar Alerta
1. Dashboard > Painel > Alerting
2. Condition: `{job="app"} |= "error"` > 10 logs/min
3. Notificação (Email, Slack, etc)

## Configuração de Promtail

### Jobs Padrão

**System Logs:**
```yaml
- job_name: system
  static_configs:
    - targets: [localhost]
      labels:
        job: system
      __path__: /var/log/**/*log
```

**Application Logs:**
```yaml
- job_name: application
  static_configs:
    - targets: [localhost]
      labels:
        job: application
      __path__: /var/log/app/**/*log
```

**Docker Logs:**
```yaml
- job_name: docker
  static_configs:
    - targets: [localhost]
      labels:
        job: docker
      __path__: /var/lib/docker/containers/*/*-json.log
```

## Integração com Stack Existente

### Fluxo Completo de Observabilidade

```
SISTEMA
  ├─ CPU/Memória/Disco
  │  └─> Node Exporter -> Prometheus -> Grafana (Métricas)
  │
  ├─ Logs
  │  └─> Promtail -> Loki -> Grafana (Logs)
  │
  └─ Dados Customizados
     └─> Telegraf -> InfluxDB -> Grafana (Séries Temporais)
```

### Exemplos de Dashboards Combinados

**Dashboard de Troubleshooting:**
- Gráfico: CPU e Memória (Prometheus)
- Tabela: Logs de erro (Loki)
- Gráfico: Taxa de requisições (InfluxDB)

## Troubleshooting

### Promtail não coleta logs
```bash
# Verificar configuração
docker logs promtail

# Testar arquivo de config
docker exec promtail promtail -config.file=/etc/promtail/config.yaml -verify-config

# Verificar permissões
ls -la /var/log/
```

### Loki não inicia
```bash
# Verificar logs
docker logs loki

# Verificar saúde
curl http://localhost:3100/ready

# Verificar espaço em disco
docker system df
```

### Grafana não visualiza logs
```bash
# Verificar conexão
docker exec grafana curl -v http://loki:3100/ready

# Testar datasource
curl 'http://localhost:3000/api/datasources/proxy/1/loki/api/v1/labels'

# Verificar logs do Grafana
docker logs grafana
```

### Alto uso de disco
```bash
# Limpar posições antigas
rm -f /tmp/positions.yaml

# Reduzir retention no Loki
# Editar loki-config.yaml e reconstruir

# Limpar volumes não usados
docker system prune
```

## Performance Tips

1. **Limitar ingestão:**
   ```yaml
   # loki-config.yaml
   limits_config:
     ingestion_rate_mb: 10
     ingestion_burst_size_mb: 20
   ```

2. **Ativar cache:**
   ```yaml
   query_range:
     cache_results: true
   ```

3. **Ajustar chunk age:**
   ```yaml
   ingester:
     max_chunk_age: 1h
   ```

4. **Usar glob patterns eficientes:**
   ```yaml
   __path__: /var/log/app/*.log  # Específico
   __path__: /var/log/**/*log    # Genérico (mais lento)
   ```

## Segurança em Produção

⚠️ **Recomendações:**

1. **Ativar autenticação no Loki:**
   ```yaml
   auth_enabled: true
   ```

2. **Usar HTTPS:**
   ```yaml
   server:
     http_tls_cert_file: /etc/loki/certs/tls.crt
     http_tls_key_file: /etc/loki/certs/tls.key
   ```

3. **Implementar retenção:**
   ```yaml
   table_manager:
     retention_enabled: true
   ```

4. **Autenticação no Grafana:**
   - Ativar LDAP/OAuth2
   - RBAC (Role-Based Access Control)

## Referências

- [Grafana Loki Docs](https://grafana.com/docs/loki/latest/)
- [LogQL Documentation](https://grafana.com/docs/loki/latest/logql/)
- [Promtail Configuration](https://grafana.com/docs/loki/latest/clients/promtail/)
- [Grafana Datasources](https://grafana.com/docs/grafana/latest/datasources/)
