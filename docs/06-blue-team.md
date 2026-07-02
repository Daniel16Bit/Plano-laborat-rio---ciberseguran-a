# Parte 6 — Ferramentas Blue Team

Atacar sem saber como o ataque aparece do outro lado é meio caminho andado. Esta parte monta a sua sala de defesa: um SIEM para centralizar e correlacionar eventos, um IDS/IPS para farejar tráfego malicioso, o Wireshark para olhar o pacote na veia, ferramentas de forense para reconstruir o que aconteceu e a mentalidade de threat hunting para caçar o que passou despercebido.

## 6.1 SIEM — Wazuh no desktop

O Wazuh é a melhor escolha para o lab: open source e, numa única plataforma, entrega SIEM, HIDS, detecção de vulnerabilidades e compliance.

| Componente | Função |
|---|---|
| Wazuh Manager | Servidor central — recebe eventos de todos os agentes |
| Wazuh Agent | Roda nas VMs e no Windows 11 — coleta logs e eventos |
| Elasticsearch | Indexa e armazena todos os logs |
| Kibana Dashboard | Visualização — alertas, gráficos, busca de eventos |
| File Integrity | Detecta alterações em arquivos críticos |
| Vuln Detection | Escaneia agentes em busca de CVEs conhecidas |

### Configurar o agente no Windows 11 (Blue Team)

1. Baixe o agente Wazuh para Windows: `https://packages.wazuh.com/4.x/windows/wazuh-agent-4.7.x-1.msi`
2. Na instalação, informe o IP do Wazuh Server VM (`192.168.100.150`).
3. O agente inicia como serviço Windows — rodando mesmo enquanto você usa o Kali.
4. Os Event IDs do Windows passam a aparecer automaticamente no painel do Wazuh.

> Quando você atacar do Kali e um login `4625` surgir no Wazuh — isso é Purple Team acontecendo diante dos seus olhos.

- [Wazuh Documentation](https://documentation.wazuh.com)
- [Wazuh Ruleset (GitHub)](https://github.com/wazuh/wazuh-ruleset) · regras e decoders para customização

## 6.2 IDS/IPS — Suricata e regras customizadas

```bash
# Instalar no Kali host
sudo apt install -y suricata

# Configurar a interface a monitorar
sudo nano /etc/suricata/suricata.yaml
# af-packet: interface: eth0  (ou a interface da LabNET)

# Atualizar regras Emerging Threats (gratuitas)
sudo suricata-update
```

Exemplo de regra customizada para detectar um Nmap SYN scan (`/etc/suricata/rules/custom.rules`):

```
alert tcp any any -> $HOME_NET any (msg:"SCAN Nmap SYN Scan";
  flags:S; threshold:type threshold,track by_src,count 20,seconds 3;
  classtype:network-scan; sid:9000001; rev:1;)
```

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml   # testar regras
tail -f /var/log/suricata/fast.log                # ver alertas
```

- [Suricata Rules Docs](https://suricata.readthedocs.io/en/latest/rules/intro.html)
- [Emerging Threats Rules](https://rules.emergingthreats.net)

## 6.3 Wireshark — filtros essenciais

| Filtro de display | O que detecta |
|---|---|
| `http.request.method == POST` | Credenciais enviadas em formulários HTTP |
| `tcp.flags.syn==1 && tcp.flags.ack==0` | SYN packets — port scanning |
| `smb2 \|\| smb` | Tráfego SMB — atividade Windows anômala |
| `dns.qry.name contains "evil"` | Consultas DNS suspeitas — possível C2 |
| `frame contains "password"` | Strings `password` em pacotes — credenciais |
| `ip.src == 192.168.100.100` | Todo tráfego do Kali atacante |
| `tcp.analysis.retransmission` | Retransmissões — pode indicar evasão |
| `kerberos` | Tráfego Kerberos — padrões anômalos |

- [Wireshark Display Filters](https://wiki.wireshark.org/DisplayFilters)
- [Malware Traffic Analysis](https://www.malware-traffic-analysis.net) · PCAPs reais de malware para praticar

## 6.4 Forense digital

### Volatility 3 — análise de memória RAM

```bash
# Capturar memória (WinPMEM no Windows, LiME no Linux)
.\winpmem_mini_x64.exe memory.dmp

# Análise com Volatility 3
python3 vol.py -f memory.dmp windows.pslist
python3 vol.py -f memory.dmp windows.pstree
python3 vol.py -f memory.dmp windows.netstat
python3 vol.py -f memory.dmp windows.dlllist --pid 1234

# Procurar injeção de código
python3 vol.py -f memory.dmp windows.malfind

# Extrair processo suspeito
python3 vol.py -f memory.dmp windows.dumpfiles --pid 1234
```

📖 [Volatility 3 Docs](https://volatility3.readthedocs.io)

## 6.5 Threat hunting

A lógica é sempre: **Hipótese → Busca nos logs → Análise → Resposta.** Você parte de uma suposição de comportamento malicioso e vai atrás de prová-la (ou descartá-la).

- Hipótese: _"O atacante pode estar usando PowerShell com encoding base64."_
  Busca no Wazuh: `event.code:4688 AND process.command_line:*encodedcommand*`
- Hipótese: _"Há movimento lateral por Pass-the-Hash."_
  Busque `EventID 4648` fora do horário comercial.

- [Sigma Rules](https://github.com/SigmaHQ/sigma) · regras de detecção agnósticas de SIEM — converta para Wazuh/Suricata
- [MITRE ATT&CK para hunting](https://attack.mitre.org) · use cada técnica como hipótese

---

⏮️ Anterior: **[Parte 5 — Red Team](05-red-team.md)** · ⏭️ Próximo: **[Parte 7 — Web Security](07-web-security.md)** · [↩︎ Índice](../README.md)
