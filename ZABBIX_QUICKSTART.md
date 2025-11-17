# 🎯 Guia Rápido: Primeiros Passos com Zabbix

## ⏱️ 5 Minutos para Começar

### Passo 1: Iniciar o Stack (1 min)

```bash
cd c:\Users\josen\Downloads\dev.docker-main
docker-compose up -d mysql zabbix-server zabbix-frontend
```

**Aguardar inicialização:**
```bash
docker-compose ps | grep zabbix
```

Quando mostrar `healthy`, continue.

### Passo 2: Acessar Interface (1 min)

```
URL: http://localhost:8080
Usuário: Admin
Senha: zabbix
```

### Passo 3: Primeira Métrica (2 min)

1. Vá para: **Configuration → Hosts**
2. Clique: **Create host**
3. Preencha:
   - Hostname: `Local Server`
   - IP Address: `127.0.0.1`
   - Port: `10050`
4. Clique: **Add**

### Passo 4: Verificar Status (1 min)

1. Vá para: **Monitoring → Hosts**
2. Procure por `Local Server`
3. Verifique se status é **Available** (verde)

✅ **Pronto!** Seu Zabbix está funcionando!

---

## 📚 Próximos Passos

### Para Monitorar Servidores Remotos

1. **Instalar Zabbix Agent**
   ```bash
   # Em cada servidor a monitorar
   apt-get install zabbix-agent
   ```

2. **Configurar Agent**
   ```
   /etc/zabbix/zabbix_agentd.conf
   
   Server=<SEU_ZABBIX_SERVER_IP>
   ServerActive=<SEU_ZABBIX_SERVER_IP>:10051
   Hostname=Nome-do-Servidor
   ```

3. **Reiniciar Agent**
   ```bash
   systemctl restart zabbix-agent
   ```

4. **Adicionar em Zabbix**
   ```
   Frontend → Configuration → Hosts → Create host
   ```

### Para Enviar Alertas

1. **Configurar Media**
   ```
   Administration → Media types → Create media type
   ```

2. **Exemplos:**
   - Email
   - Slack
   - Discord
   - SMS (Twilio)
   - Webhook customizado

### Para Visualizar em Grafana

1. **Adicionar Datasource**
   ```
   Grafana → Configuration → Data Sources → Add
   Type: Zabbix
   URL: http://zabbix-server:10051
   ```

2. **Criar Dashboard**
   ```
   + → Dashboard → Add panel
   Datasource: Zabbix
   Selecionar métricas
   ```

---

## 🔍 Verificação de Saúde

### Status Rápido

```bash
# Verificar se todos estão rodando
docker-compose ps | grep -E "zabbix|mysql"

# Ver logs em tempo real
docker-compose logs -f zabbix-server

# Testar conexão
docker-compose exec zabbix-server curl http://zabbix-server:10051
```

### Checklist

- [ ] MySQL rodando
- [ ] Zabbix Server rodando (healthy)
- [ ] Zabbix Frontend respondendo (http://localhost:8080)
- [ ] Frontend consegue conectar ao Server
- [ ] Consegue fazer login (Admin/zabbix)

---

## 📊 Dashboards Básicos

### Dashboard 1: System Overview

**Painéis:**
1. CPU Load
2. Memory Usage
3. Disk Usage
4. Uptime
5. Network Traffic

**Como criar:**
```
Monitoring → Dashboards → Create dashboard
+ Add panel
Selecionar host e métrica
```

### Dashboard 2: Alertas

**Painéis:**
1. Recent Problems
2. Trigger Status
3. Alert History

### Dashboard 3: Performance

**Painéis:**
1. Response Time
2. Data Collection
3. Database Performance

---

## 🛠️ Customizações Comuns

### Aumentar Período de Retenção

```yaml
# Editar config/zabbix_server.conf
HousekeeperFrequency=1
MaxHousekeeperDelete=10000
```

### Alterar Linguagem

```
Profile (canto superior direito) → Language → Português
```

### Adicionar Logo Customizada

```
Administration → General → GUI
Logo URL: https://seu-logo.png
```

---

## 🆘 Problemas Comuns

### "Cannot connect to Zabbix agent"

**Solução:**
1. Verificar se Agent está instalado
2. Verificar firewall (porta 10050)
3. Verificar configuração do Agent
4. Reiniciar Agent: `systemctl restart zabbix-agent`

### "Database error"

**Solução:**
1. Verificar MySQL: `docker-compose ps mysql`
2. Reiniciar MySQL: `docker-compose restart mysql`
3. Verificar logs: `docker-compose logs mysql`

### "Frontend não abre"

**Solução:**
1. Verificar se Zabbix está rodando: `docker-compose ps`
2. Reiniciar: `docker-compose restart zabbix-frontend`
3. Limpar cache do navegador

---

## 📈 Monitoramento Avançado

### Triggers (Alertas)

```
Configuration → Triggers → Create trigger

Exemplo: CPU > 80%
Expression: {Local Server:system.cpu.load.avg(5m)}>80
```

### Ações (Ações automáticas)

```
Configuration → Actions → Create action

Quando: Trigger dispara
Fazer: Enviar email/webhook/etc
```

### Macros

```
Administration → General → Macros

{$CRITICAL_CPU}=85
{$WARNING_MEM}=75
```

---

## 🎓 Recursos de Aprendizado

### Documentação Oficial
- https://www.zabbix.com/documentation/6.0/

### Vídeos
- Zabbix Channel no YouTube

### Comunidade
- Zabbix Forum
- Stack Overflow (tag: zabbix)

---

## ✅ Checklist Final

### Configuração Inicial
- [ ] Docker-compose up executado
- [ ] MySQL conectado
- [ ] Zabbix Server healthy
- [ ] Frontend acessível

### Primeira Métrica
- [ ] Host criado
- [ ] Agent conectado
- [ ] Dados sendo coletados
- [ ] Métricas visíveis

### Produção Ready
- [ ] Backups configurados
- [ ] Alertas configurados
- [ ] Senhas alteradas
- [ ] Firewall configurado

---

## 🚀 Próximo Nível

### Integração com Prometheus
```bash
# Adicionar exportador Prometheus
docker-compose up -d zabbix-prometheus-exporter
```

### Integração com Grafana
```
Grafana → Data Sources → Add Zabbix
```

### Backup Automático
```
Scheduling backup de MySQL
```

### Clustering
```
Zabbix Server em cluster para HA
```

---

**Parabéns! Você está pronto para começar com Zabbix! 🎉**

Para mais detalhes, consulte:
- `ZABBIX_INTEGRATION.md` - Integração completa
- `compose/zabbix-server/readme.md` - Documentação técnica
- Frontend Zabbix em `http://localhost:8080`
