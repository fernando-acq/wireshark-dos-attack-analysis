# 🦈 Wireshark DoS Attack Analysis

Análise de ataque de Negação de Serviço (DoS) do tipo SYN Flood utilizando Wireshark para captura e investigação de tráfego de rede malicioso.

## 📋 Descrição

Este projeto documenta a análise completa de um ataque DoS (Denial of Service) do tipo SYN Flood em uma rede corporativa. Utilizando a ferramenta **Wireshark**, foi possível capturar, analisar e diagnosticar o tráfego malicioso que causou a indisponibilidade do serviço web da organização.

O relatório demonstra a capacidade de identificar padrões de ataque, interpretar logs de rede, entender o funcionamento do protocolo TCP e propor medidas efetivas de mitigação e prevenção.

## 🎯 Objetivos do Projeto

- Identificar o tipo de ataque através da análise de tráfego de rede
- Compreender o mecanismo de funcionamento do ataque SYN Flood
- Diagnosticar o impacto do ataque na infraestrutura
- Propor medidas de proteção e mitigação
- Documentar o incidente seguindo boas práticas de resposta

## 🛠️ Tecnologias Utilizadas

- **Wireshark** - Captura e análise de pacotes
- **TCP/IP** - Protocolos de rede
- **HTTP/HTTPS** - Protocolos de aplicação
- Análise de logs de rede
- Resposta a incidentes

## 📁 Estrutura do Projeto

```
wireshark-dos-attack-analysis/
│
├── README.md                        # Documentação do projeto
├── incident_report.md               # Relatório completo do incidente
├── wireshark_analysis_guide.md      # Guia de análise no Wireshark
└── mitigation_strategies.md         # Estratégias de mitigação
```

## 🚨 Resumo do Incidente

### Tipo de Ataque Identificado

**Ataque DoS (Denial of Service) - SYN Flood**

A análise do tráfego de rede revelou múltiplas conexões repetitivas em alta velocidade, todas provenientes do mesmo endereço IP. O ataque utiliza o protocolo TCP e consiste no envio massivo de pacotes SYN sem completar o handshake de três vias, esgotando os recursos do servidor.

### Sintomas Observados

- Mensagens de "Time-out" para usuários legítimos
- Indisponibilidade do site corporativo
- Alto consumo de recursos do servidor
- Impossibilidade de estabelecer novas conexões TCP

### Protocolo Afetado

**TCP (Transmission Control Protocol)**

O ataque explora o processo de handshake TCP de três vias:
1. Cliente envia SYN
2. Servidor responde com SYN-ACK
3. Cliente deveria enviar ACK (mas não envia no ataque)

## 🔍 Análise Técnica

### TCP Handshake Normal

```
Cliente          Servidor
   |                |
   |------ SYN ---->|  (Passo 1: Solicitação de conexão)
   |<--- SYN-ACK ---|  (Passo 2: Confirmação do servidor)
   |------ ACK ---->|  (Passo 3: Confirmação do cliente)
   |                |
   [Conexão estabelecida]
```

### Ataque SYN Flood

```
Atacante         Servidor
   |                |
   |------ SYN ---->|  (Múltiplos pacotes SYN)
   |<--- SYN-ACK ---|  (Servidor aguarda ACK)
   |                |  (ACK nunca chega)
   |------ SYN ---->|  (Mais pacotes SYN)
   |<--- SYN-ACK ---|  (Servidor mantém conexões pendentes)
   |                |  (Recursos do servidor esgotados)
   |------ SYN ---->|
   |                |
   [Servidor sobrecarregado]
```

### Evidências Capturadas no Wireshark

**Padrões identificados:**

- **Verde:** Conexões TCP normais (handshake completo)
- **Amarelo:** Conexões TCP falhadas devido à sobrecarga
- **Vermelho:** Pacotes SYN maliciosos (ataque em andamento)

**Exemplo de log capturado:**

```
79  7.360768  198.51.100.22  192.0.2.1  TCP  6345->443 [SYN] Seq=0 Win=5792 Len=0
80  7.380773  192.0.2.1  198.51.100.7  TCP  443->42584 [RST, ACK] Seq=1 Win=5792 Len=120
81  7.380878  203.0.113.0  192.0.2.1  TCP  54770->443 [SYN] Seq=0 Win=5792 Len=0
82  7.383879  203.0.113.0  192.0.2.1  TCP  54770->443 [SYN] Seq=0 Win=5792 Len=0
83  7.482754  192.0.2.1  203.0.113.0  TCP  443->54770 [RST, ACK] Seq=1 Win=5792 Len=0
```

**Análise:**
- Múltiplos pacotes [SYN] sem pacotes [ACK] correspondentes
- Servidor responde com [RST, ACK] tentando resetar conexões
- Alto volume de tentativas em curto período de tempo
- Todas as conexões incompletas

## 🛡️ Medidas de Proteção Recomendadas

### 1. SYN Cookies

Técnica que permite ao servidor responder a pacotes SYN sem alocar recursos até receber o ACK final, protegendo contra esgotamento de recursos.

### 2. Firewall e Filtragem de Tráfego

- Configurar regras de rate limiting para conexões SYN
- Bloquear endereços IP suspeitos
- Implementar listas de bloqueio (blacklists)

### 3. IDS/IPS (Sistema de Detecção/Prevenção de Intrusão)

- Detectar padrões de ataque em tempo real
- Bloquear automaticamente tráfego malicioso
- Alertar equipe de segurança sobre anomalias

### 4. Proteção DoS/DDoS Baseada em Nuvem

- Cloudflare, AWS Shield, Akamai
- Absorção de tráfego malicioso antes de atingir o servidor
- Distribuição de carga e mitigação em larga escala

### 5. Monitoramento Contínuo

- Análise de logs em tempo real
- Alertas automáticos para tráfego anômalo
- Dashboards de segurança (SIEM)

## 📊 Impacto do Incidente

### Técnico
- Indisponibilidade do serviço web
- Sobrecarga do servidor
- Falha nas conexões legítimas

### Negócios
- Perda de receita durante a interrupção
- Impacto na reputação da empresa
- Insatisfação de clientes

## 🔒 Aplicações em Cibersegurança

Este projeto demonstra competências essenciais para:

- **SOC Analyst:** Monitoramento e detecção de ataques de rede
- **Incident Response:** Análise e resposta a incidentes de segurança
- **Network Security:** Proteção de infraestrutura de rede
- **Threat Analysis:** Identificação de padrões de ataque
- **Security Engineering:** Implementação de controles de segurança

## 📚 Conceitos Aplicados

- Análise de tráfego de rede com Wireshark
- Protocolo TCP e handshake de três vias
- Ataques DoS e suas variações
- Resposta a incidentes de segurança
- Documentação técnica de incidentes
- Medidas de mitigação e prevenção

## 🎓 Aprendizados

- Identificação de padrões de ataque através de análise de pacotes
- Interpretação de logs do Wireshark
- Compreensão profunda do protocolo TCP
- Elaboração de relatórios de incidente
- Proposta de soluções de segurança efetivas

## 🤝 Contribuições

Sugestões e melhorias são bem-vindas! Sinta-se à vontade para abrir issues ou pull requests.

## 📝 Licença

Este projeto está sob a licença MIT. Veja o arquivo LICENSE para mais detalhes.

## 👤 Autor

**Fernando Acquesta**
- GitHub: [@fernando-acq](https://github.com/fernando-acq)
- LinkedIn: [Fernando Acquesta](https://www.linkedin.com/in/fernando-acquesta-cybersecurity)

---

⭐ Se este projeto ajudou você a entender melhor ataques DoS e análise de rede, considere dar uma estrela no repositório!