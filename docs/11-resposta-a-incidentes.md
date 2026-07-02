# Parte 11 — Cenários de defesa e resposta a incidentes

Agora é o lado do defensor. A graça de ter feito os ataques da [Parte 10](10-cenarios-de-ataque.md) é poder voltar a eles enxergando tudo pelo Wazuh, pelo Wireshark e pela memória da máquina. Esta parte estrutura essa investigação usando o framework que o mercado adota — o NIST SP 800-61 — e mostra, na prática, como caçar os rastros do EternalBlue.

## 11.1 Framework NIST SP 800-61 — IR no seu lab

| Fase | Atividades no laboratório |
|---|---|
| 1. Preparação | Wazuh configurado, regras Suricata ativas, baselines estabelecidos |
| 2. Detecção | Wazuh alerta, Suricata detecta, Wireshark captura — você analisa |
| 3. Contenção | Snapshot da VM, isolar da TargetNET, preservar evidências |
| 4. Análise | Volatility no dump de RAM, Autopsy no disco, correlação de logs |
| 5. Erradicação | Identificar e remover backdoor, revogar credenciais, aplicar patch |
| 6. Recuperação | Restaurar VM de snapshot limpo, validar, monitoramento extra |
| 7. Lições aprendidas | Relatório: timeline, IOCs, causa raiz, como detectar na próxima vez |

📖 [NIST SP 800-61r2](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final) · guia completo e gratuito

## 11.2 Investigação forense do EternalBlue — passo a passo

Continuação do Cenário 2 da Parte 10, agora pelo lado defensivo.

### No Wazuh (Kibana Dashboard)
```
# Filtros de busca
data.win.system.eventID: 4625 AND agent.name: MSF3-Windows
rule.groups: smb AND rule.level: >= 10
```
```bash
# Via linha de comando no Wazuh Manager
grep 'ms17-010' /var/ossec/logs/alerts/alerts.log
```

### Análise de tráfego
```bash
# No Wireshark, filtre o tráfego do incidente:
tcp.port == 445 && ip.src == 192.168.100.100
# Procure: SMB2 Tree Connect → IPC$ → payload shellcode

# Exportar o PCAP do período do incidente
tshark -r captura.pcap -Y 'ip.addr == 192.168.200.20 && tcp.port == 445'
```

### Forense de memória
```bash
python3 vol.py -f memory_msf3win.dmp windows.pstree
# Procure: cmd.exe ou powershell.exe com parent process suspeito

python3 vol.py -f memory_msf3win.dmp windows.netstat
# Procure: conexão para 192.168.100.100 na porta 4444

python3 vol.py -f memory_msf3win.dmp windows.malfind
# Detecta injeção de código (meterpreter na memória)
```

## 11.3 Ferramentas de IR adicionais

- [TheHive Project](https://thehive-project.org) · plataforma de IR open source — gerencia casos e IOCs
- [MISP](https://www.misp-project.org) · compartilhamento de threat intel e IOCs
- [Velociraptor](https://docs.velociraptor.app) · coleta forense em escala — excelente para o lab

---

⏮️ Anterior: **[Parte 10 — Cenários de ataque](10-cenarios-de-ataque.md)** · ⏭️ Próximo: **[Parte 12 — Exercícios progressivos](12-exercicios-progressivos.md)** · [↩︎ Índice](../README.md)
