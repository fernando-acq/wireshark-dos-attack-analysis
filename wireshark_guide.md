# 🔍 Guia de Análise com Wireshark

Guia prático para análise de tráfego de rede e detecção de ataques DoS usando Wireshark.

---

## 📥 Instalação do Wireshark

### Linux
```bash
sudo apt update
sudo apt install wireshark
```

### Windows/macOS
Baixe em: https://www.wireshark.org/download.html

---

## 🚀 Iniciando uma Captura

### 1. Selecionar Interface de Rede
- Abra o Wireshark
- Selecione a interface de rede ativa (ex: eth0, wlan0)
- Clique duas vezes para iniciar captura

### 2. Iniciar Captura
```
Menu: Capture -> Start
Atalho: Ctrl + E
```

### 3. Parar Captura
```
Menu: Capture -> Stop
Atalho: Ctrl + E
```

---

## 🔎 Filtros Úteis no Wireshark

### Filtros de Protocolo
```
tcp                 # Apenas tráfego TCP
udp                 # Apenas tráfego UDP
http                # Apenas HTTP
https               # Apenas HTTPS (TLS)
dns                 # Apenas DNS
```

### Filtros de Endereço IP
```
ip.addr == 192.168.1.1          # Tráfego de/para este IP
ip.src == 192.168.1.1           # Origem deste IP
ip.dst == 192.168.1.1           # Destino para este IP
```

### Filtros de Porta
```
tcp.port == 443                 # Tráfego na porta 443
tcp.srcport == 443              # Origem porta 443
tcp.dstport == 443              # Destino porta 443
```

### Filtros TCP Específicos
```
tcp.flags.syn == 1              # Pacotes SYN
tcp.flags.syn == 1 and tcp.flags.ack == 0   # Apenas SYN (sem ACK)
tcp.flags.reset == 1            # Pacotes RST
tcp.analysis.retransmission     # Retransmissões
```

### Filtros Combinados
```
tcp.flags.syn == 1 and ip.src == 192.168.1.100    # SYN de IP específico
tcp.port == 443 and tcp.flags.syn == 1            # SYN na porta 443
```

---

## 🚨 Identificando Ataque SYN Flood

### Sinais de Ataque SYN Flood

1. **Alto volume de pacotes SYN**
   - Filtro: `tcp.flags.syn == 1 and tcp.flags.ack == 0`
   - Muitos pacotes SYN sem ACK correspondente

2. **Mesmo IP de origem**
   - Filtro: `ip.src == [IP_SUSPEITO]`
   - Múltiplas conexões do mesmo endereço

3. **Pacotes RST do servidor**
   - Filtro: `tcp.flags.reset == 1`
   - Servidor tentando resetar conexões

4. **Padrão de cores no Wireshark**
   - Verde: Conexões normais
   - Amarelo: Conexões falhadas
   - Vermelho: Tráfego suspeito

### Análise Estatística

**Statistics -> Protocol Hierarchy**
- Visualiza distribuição de protocolos
- Identifica anomalias de volume

**Statistics -> Conversations**
- Mostra conversas entre hosts
- Identifica IPs com alto volume de conexões

**Statistics -> Endpoints**
- Lista todos os endpoints
- Ordena por pacotes/bytes enviados

---

## 📊 Exemplo de Análise de SYN Flood

### Passo 1: Filtrar Pacotes SYN
```
tcp.flags.syn == 1 and tcp.flags.ack == 0
```

### Passo 2: Verificar IPs de Origem
```
Statistics -> Conversations -> TCP
```
Ordenar por pacotes para identificar IP com maior volume.

### Passo 3: Analisar Handshake Incompleto
```
tcp.stream eq [número_do_stream]
```
Follow TCP Stream para ver se handshake foi completado.

### Passo 4: Identificar Padrão Temporal
```
Statistics -> I/O Graph
```
Visualizar picos de tráfego ao longo do tempo.

---

## 🎨 Interpretação das Cores

### Regras de Coloração Padrão

- **Verde Claro:** Tráfego HTTP
- **Azul Claro:** Tráfego DNS
- **Preto:** Tráfego TCP normal
- **Vermelho:** Erros e problemas
- **Amarelo:** Avisos e alertas

### Personalizar Cores

```
View -> Coloring Rules
```
Criar regras personalizadas para destacar padrões específicos.

---

## 📝 Exportando Dados

### Exportar Pacotes Capturados
```
File -> Export Specified Packets
```

### Exportar como Texto
```
File -> Export Packet Dissections -> As Plain Text
```

### Exportar Objetos HTTP
```
File -> Export Objects -> HTTP
```

---

## 🔍 Campos Importantes no TCP

### Cabeçalho TCP

| Campo | Descrição |
|-------|-----------|
| Source Port | Porta de origem |
| Destination Port | Porta de destino |
| Sequence Number | Número de sequência |
| Acknowledgment | Número de confirmação |
| Flags | SYN, ACK, FIN, RST, etc. |
| Window Size | Tamanho da janela |

### Flags TCP

| Flag | Descrição |
|------|-----------|
| SYN | Sincronização (início de conexão) |
| ACK | Confirmação |
| FIN | Finalização de conexão |
| RST | Reset de conexão |
| PSH | Push de dados |
| URG | Dados urgentes |

---

## 💡 Dicas de Análise

### Para Investigar Ataques DoS

1. Procure por padrões repetitivos
2. Identifique IPs com volume anormal
3. Verifique handshakes incompletos
4. Analise intervalos de tempo entre pacotes
5. Compare com tráfego legítimo

### Para Análise Eficiente

1. Use filtros para focar no relevante
2. Salve filtros frequentes
3. Use cores para destacar padrões
4. Exporte dados para análise adicional
5. Documente suas descobertas

---

## 🎓 Comandos de Display Filter

### Operadores Lógicos
```
and         # E lógico
or          # OU lógico
not         # NÃO lógico
==          # Igual
!=          # Diferente
>           # Maior que
<           # Menor que
```

### Exemplos Práticos
```
tcp.flags.syn == 1 and not tcp.flags.ack == 1
ip.addr == 192.168.1.0/24
tcp.port == 80 or tcp.port == 443
http.request.method == "GET"
```

---

## 📚 Recursos Adicionais

- Documentação oficial: https://www.wireshark.org/docs/
- Wiki do Wireshark: https://wiki.wireshark.org/
- Display Filters: https://wiki.wireshark.org/DisplayFilters

---

**Nota:** Este guia foca nos comandos e conceitos utilizados no projeto de análise do ataque SYN Flood.