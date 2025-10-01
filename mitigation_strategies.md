# 🛡️ Estratégias de Mitigação contra Ataques DoS/DDoS

Guia completo de medidas de proteção e prevenção contra ataques de negação de serviço.

---

## 1. SYN Cookies

### O que são?

SYN Cookies são uma técnica que permite ao servidor responder a pacotes SYN sem alocar recursos até receber o ACK final, protegendo contra esgotamento de recursos durante ataques SYN Flood.

### Como funciona?

1. Servidor recebe pacote SYN
2. Em vez de alocar recursos, servidor calcula um cookie baseado em informações da conexão
3. Servidor envia SYN-ACK com o cookie no campo de número de sequência
4. Se cliente responder com ACK válido (incluindo o cookie), servidor aloca recursos

### Vantagens

- Não consome recursos para conexões incompletas
- Proteção transparente para o cliente
- Mantém compatibilidade com TCP padrão

### Como implementar

**Linux:**
```bash
# Habilitar SYN Cookies
echo 1 > /proc/sys/net/ipv4/tcp_syncookies

# Persistente (adicionar em /etc/sysctl.conf)
net.ipv4.tcp_syncookies = 1
```

---

## 2. Firewall e Filtragem de Tráfego

### Rate Limiting

Limitar número de conexões SYN por segundo de cada IP.

**iptables (Linux):**
```bash
# Limitar novas conexões a 5 por segundo por IP
iptables -A INPUT -p tcp --syn -m limit --limit 5/s -j ACCEPT
iptables -A INPUT -p tcp --syn -j DROP
```

### Bloqueio de IPs Suspeitos

**iptables:**
```bash
# Bloquear IP específico
iptables -A INPUT -s 198.51.100.22 -j DROP

# Bloquear range de IPs
iptables -A INPUT -s 198.51.100.0/24 -j DROP
```

### Regras de Firewall Avançadas

```bash
# Permitir apenas conexões estabelecidas
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Limitar conexões simultâneas por IP
iptables -A INPUT -p tcp --syn -m connlimit --connlimit-above 10 -j REJECT
```

---

## 3. IDS/IPS (Sistema de Detecção/Prevenção de Intrusão)

### Snort

Sistema de detecção de intrusão open source.

**Regra para detectar SYN Flood:**
```
alert tcp any any -> $HOME_NET any (flags: S; threshold: type both, track by_src, count 100, seconds 10; msg:"Possible SYN Flood Attack"; sid:1000001;)
```

### Suricata

IDS/IPS open source de alto desempenho.

**Configuração básica:**
```yaml
# Detectar alto volume de SYN
alert tcp any any -> $HOME_NET any (flags:S; threshold: type threshold, track by_src, count 50, seconds 10; msg:"SYN Flood Detected"; sid:2000001;)
```

### Fail2Ban

Ferramenta para banir IPs baseado em logs.

**Configuração para SSH:**
```ini
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
```

---

## 4. Proteção DoS/DDoS Baseada em Nuvem

### Cloudflare

**Recursos:**
- Absorção de tráfego DDoS
- CDN global
- WAF (Web Application Firewall)
- Rate limiting automático

**Como funciona:**
1. Tráfego passa primeiro pelos servidores Cloudflare
2. Tráfego malicioso é filtrado na borda
3. Apenas tráfego legítimo chega ao servidor

### AWS Shield

**Recursos:**
- Shield Standard (gratuito, proteção básica)
- Shield Advanced (proteção avançada contra DDoS)
- Integração com CloudFront, Route 53, ELB

### Akamai Kona Site Defender

**Recursos:**
- Proteção DDoS em camada de rede e aplicação
- Mitigação baseada em nuvem
- Análise de tráfego em tempo real

---

## 5. Configurações do Servidor

### Ajustes no Kernel Linux

**Aumentar fila de conexões pendentes:**
```bash
# Aumentar backlog de conexões SYN
echo 1024 > /proc/sys/net/ipv4/tcp_max_syn_backlog

# Reduzir tempo de espera para SYN-ACK
echo 3 > /proc/sys/net/ipv4/tcp_synack_retries

# Habilitar SYN Cookies
echo 1 > /proc/sys/net/ipv4/tcp_syncookies
```

**Para tornar permanente (adicionar em /etc/sysctl.conf):**
```
net.ipv4.tcp_max_syn_backlog = 1024
net.ipv4.tcp_synack_retries = 3
net.ipv4.tcp_syncookies = 1
```

### Configurações de Apache

```apache
# Limitar conexões simultâneas
<IfModule mpm_worker_module>
    MaxRequestWorkers 150
    ServerLimit 150
</IfModule>

# Timeout de requisição
Timeout 30
```

### Configurações de Nginx

```nginx
# Limitar conexões por IP
limit_conn_zone $binary_remote_addr zone=addr:10m;
limit_conn addr 10;

# Limitar taxa de requisições
limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;
limit_req zone=one burst=20;
```

---

## 6. Monitoramento Contínuo

### Ferramentas de Monitoramento

**Netstat - Visualizar conexões:**
```bash
# Ver conexões TCP
netstat -ant

# Contar conexões SYN_RECV (indicativo de ataque)
netstat -ant | grep SYN_RECV | wc -l

# Ver IPs com mais conexões
netstat -ntu | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -n
```

**ss - Alternativa moderna ao netstat:**
```bash
# Ver estatísticas de sockets
ss -s

# Contar conexões SYN-RECV
ss -ant | grep SYN-RECV | wc -l
```

**tcpdump - Captura de pacotes:**
```bash
# Capturar apenas pacotes SYN
tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0'

# Capturar e salvar em arquivo
tcpdump -i eth0 -w ataque.pcap
```

### Alertas Automáticos

**Script de monitoramento básico:**
```bash
#!/bin/bash
# Alertar se houver muitas conexões SYN_RECV

THRESHOLD=100
COUNT=$(netstat -ant | grep SYN_RECV | wc -l)

if [ $COUNT -gt $THRESHOLD ]; then
    echo "ALERTA: Possível ataque SYN Flood detectado!"
    echo "Conexões SYN_RECV: $COUNT"
    # Enviar email ou notificação
fi
```

---

## 7. Plano de Resposta a Incidentes

### Fase 1: Detecção

1. Monitorar métricas de rede continuamente
2. Configurar alertas para anomalias
3. Analisar logs regularmente

### Fase 2: Análise

1. Capturar tráfego com Wireshark/tcpdump
2. Identificar padrão de ataque
3. Localizar origem do ataque
4. Documentar evidências

### Fase 3: Contenção

1. Ativar SYN Cookies
2. Bloquear IPs maliciosos no firewall
3. Ativar proteção DDoS em nuvem se disponível
4. Aumentar recursos temporariamente se necessário

### Fase 4: Erradicação

1. Implementar regras permanentes de firewall
2. Atualizar configurações do servidor
3. Aplicar patches de segurança

### Fase 5: Recuperação

1. Verificar estabilidade do serviço
2. Monitorar por atividade residual
3. Restaurar operações normais

### Fase 6: Lições Aprendidas

1. Documentar o incidente
2. Revisar resposta e melhorias
3. Atualizar procedimentos
4. Treinar equipe

---

## 8. Checklist de Proteção

### Imediato (Implementar Agora)

- [ ] Habilitar SYN Cookies
- [ ] Configurar rate limiting no firewall
- [ ] Implementar monitoramento básico
- [ ] Documentar baseline de tráfego normal

### Curto Prazo (1-2 semanas)

- [ ] Implementar IDS/IPS
- [ ] Configurar alertas automáticos
- [ ] Criar plano de resposta a incidentes
- [ ] Treinar equipe em detecção de ataques

### Médio Prazo (1-3 meses)

- [ ] Contratar proteção DDoS em nuvem
- [ ] Implementar SIEM para correlação de eventos
- [ ] Realizar testes de penetração
- [ ] Revisar e atualizar políticas de segurança

### Longo Prazo (Contínuo)

- [ ] Monitoramento 24/7
- [ ] Revisões trimestrais de segurança
- [ ] Treinamento contínuo da equipe
- [ ] Atualização constante de defesas

---

## 9. Comparação de Soluções

| Solução | Custo | Eficácia | Complexidade | Recomendado Para |
|---------|-------|----------|--------------|------------------|
| SYN Cookies | Grátis | Média | Baixa | Todos os servidores |
| Firewall (iptables) | Grátis | Média-Alta | Média | Pequenas empresas |
| IDS/IPS | Baixo-Médio | Alta | Alta | Médias empresas |
| Proteção em Nuvem | Médio-Alto | Muito Alta | Baixa | Grandes empresas |

---

## 10. Recursos Adicionais

### Documentação

- iptables: https://www.netfilter.org/documentation/
- Snort: https://www.snort.org/documents
- Cloudflare: https://www.cloudflare.com/ddos/

### Ferramentas

- Wireshark: https://www.wireshark.org/
- Suricata: https://suricata.io/
- Fail2Ban: https://www.fail2ban.org/

---

**Nota:** Este guia apresenta estratégias baseadas nos conceitos aplicados no projeto de análise do ataque SYN Flood. A implementação deve ser adaptada ao ambiente específico de cada organização.