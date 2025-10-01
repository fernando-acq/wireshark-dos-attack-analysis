# 📄 Relatório de Incidente de Segurança

**Tipo de Incidente:** Ataque de Negação de Serviço (DoS)  
**Data da Análise:** Janeiro 2025  
**Ferramenta Utilizada:** Wireshark  
**Analista:** Fernando Acquesta

---

## 1. Resumo Executivo

Foi identificado um ataque de Negação de Serviço (DoS) do tipo **SYN Flood** que causou a indisponibilidade do site corporativo. A análise de tráfego de rede utilizando Wireshark revelou múltiplas tentativas de conexão TCP incompletas provenientes de endereços IP suspeitos, resultando em esgotamento dos recursos do servidor e impossibilidade de acesso por usuários legítimos.

---

## 2. Tipo de Ataque Identificado

### Ataque DoS - SYN Flood

A análise do tráfego de rede com a ferramenta **Wireshark** revelou múltiplas conexões repetitivas e em alta velocidade, todas provenientes do mesmo usuário. Isso caracteriza um ataque **DoS (Denial of Service)**.

**Protocolo envolvido:** TCP

**Padrão de ataque:** Inundação SYN (SYN Flood), no qual o invasor envia uma grande quantidade de pacotes **SYN** sem completar o handshake de três vias.

**Resultado:** O servidor mantém inúmeras conexões pendentes, consumindo seus recursos e impedindo conexões legítimas.

**Evidência:** Mensagem de erro **"Time-out"** indicando que as solicitações legítimas estão falhando devido à sobrecarga no servidor.

---

## 3. Mecanismo do Ataque

### Handshake TCP Normal

Quando um visitante acessa o site, ocorre o handshake **TCP**, um processo essencial do protocolo TCP, responsável por estabelecer conexões confiáveis entre dispositivos. Esse processo acontece em três etapas:

1. O dispositivo envia um pacote **SYN** ao servidor para iniciar a conexão
2. O servidor responde com um **SYN-ACK** para confirmar o recebimento
3. O dispositivo finaliza o processo enviando um **ACK**, estabelecendo a conexão

Após a conexão estabelecida, o dispositivo solicita a página web **"sales.html"** via requisição **"GET"**, e o servidor responde normalmente com **"OK"**.

### Como o Ataque Funciona

No ataque **SYN Flood**, um agente malicioso envia um grande volume de pacotes **SYN** sem completar o handshake. 

**Consequências:**
- Esgotamento dos recursos do servidor
- Servidor torna-se inoperante
- Impedimento de acessos legítimos
- Impactos financeiros e operacionais devido à indisponibilidade do serviço

---

## 4. Evidências Técnicas

### Análise dos Logs do Wireshark

A captura de pacotes mostra múltiplos pacotes **SYN** enviados ao servidor sem a resposta **ACK**, caracterizando um ataque **SYN Flood**. O atacante tenta sobrecarregar o sistema ao iniciar inúmeras conexões sem concluí-las.

### Interpretação das Cores nos Logs

- **Verde:** Conexão normal de TCP handshake
- **Amarelo:** Conexão normal de TCP falhada devido ao excesso de pedidos de sincronização
- **Vermelho:** O ataque em atividade (pacotes SYN de inundação)

### Exemplo de Captura de Pacotes

```
79  7.360768  198.51.100.22  192.0.2.1  TCP  6345->443 [SYN] Seq=0 Win=5792 Len=0
80  7.380773  192.0.2.1  198.51.100.7  TCP  443->42584 [RST, ACK] Seq=1 Win=5792 Len=120
81  7.380878  203.0.113.0  192.0.2.1  TCP  54770->443 [SYN] Seq=0 Win=5792 Len=0
82  7.383879  203.0.113.0  192.0.2.1  TCP  54770->443 [SYN] Seq=0 Win=5792 Len=0
83  7.482754  192.0.2.1  203.0.113.0  TCP  443->54770 [RST, ACK] Seq=1 Win=5792 Len=0
```

**Observações:**
- Várias conexões sendo iniciadas (pacotes [SYN])
- Ausência de pacotes ACK de conclusão de handshake
- Servidor respondendo com [RST, ACK] para resetar conexões
- Evidência clara do ataque SYN Flood

---

## 5. Impacto do Incidente

### Impacto Técnico
- Indisponibilidade do site corporativo
- Sobrecarga dos recursos do servidor
- Falha nas conexões de usuários legítimos
- Mensagens de "Time-out" generalizadas

### Impacto nos Negócios
- Perda de receita durante período de indisponibilidade
- Danos à reputação da empresa
- Insatisfação de clientes
- Possível perda de oportunidades de negócio

---

## 6. Medidas de Mitigação Recomendadas

### 6.1 SYN Cookies

Utilizar **SYN Cookies** para lidar com conexões suspeitas sem consumir recursos do servidor. Esta técnica permite ao servidor responder a pacotes SYN sem alocar recursos até receber o ACK final.

### 6.2 Firewall e IDS/IPS

Configurar **Firewalls e IDS/IPS** para:
- Bloquear tráfego anômalo
- Detectar padrões de ataque
- Implementar rate limiting para conexões SYN
- Criar listas de bloqueio de IPs suspeitos

### 6.3 Proteção DoS/DDoS Baseada em Nuvem

Implementar **Proteção DoS/DDoS baseada na nuvem** para:
- Mitigar ataques em larga escala
- Absorver tráfego malicioso antes de atingir o servidor
- Distribuir carga entre múltiplos pontos de presença

**Soluções recomendadas:**
- Cloudflare
- AWS Shield
- Akamai Kona Site Defender

### 6.4 Monitoramento Contínuo

**Monitorar continuamente** o tráfego de rede para:
- Identificar atividades suspeitas antes que causem impacto
- Analisar logs em tempo real
- Gerar alertas automáticos para anomalias
- Manter dashboards de segurança atualizados

---

## 7. Ações Imediatas Tomadas

1. Identificação do ataque através de análise de logs
2. Documentação das evidências
3. Bloqueio temporário dos IPs maliciosos
4. Notificação à equipe de TI
5. Elaboração deste relatório de incidente

---

## 8. Recomendações para Prevenção Futura

1. Implementar todas as medidas de mitigação listadas na seção 6
2. Realizar testes de penetração periódicos
3. Treinar equipe em resposta a incidentes
4. Estabelecer processo de monitoramento 24/7
5. Criar plano de continuidade de negócios para cenários de ataque
6. Revisar e atualizar políticas de segurança regularmente

---

## 9. Conclusão

O incidente foi causado por um ataque DoS do tipo SYN Flood que explorou o processo de handshake TCP para esgotar os recursos do servidor. A análise detalhada usando Wireshark permitiu identificar o padrão de ataque e propor medidas efetivas de mitigação. A implementação das recomendações deste relatório fortalecerá significativamente a postura de segurança da organização contra futuros ataques similares.

---

**Relatório elaborado por:** Fernando Acquesta  
**Data:** Janeiro 2025  
**Ferramenta:** Wireshark
