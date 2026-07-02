# Parte 12 — Exercícios progressivos: do iniciante ao avançado

Aprender segurança é como levantar peso: você aumenta a carga aos poucos. Estes exercícios estão organizados em três níveis. No começo, siga os guias sem culpa. No meio, comece a encadear técnicas sozinho. No fim, opere sem walkthrough e documente como um profissional. Não pule etapas — a base construída no Nível 1 é o que sustenta o Nível 3.

## 🟢 Nível 1 — Iniciante (semanas 1–4)

**Meta:** familiarizar-se com as ferramentas. Siga guias. Ainda não se cobre por fazer tudo sozinho.

### Ex 1.1 — Primeiro Nmap
Mapeie toda a TargetNET e liste hosts, portas abertas e serviços.
```bash
nmap -sV -sC 192.168.200.0/24 -oN meu_primeiro_scan.txt
```
_O que aprender:_ como o Nmap enumera serviços e identifica o OS.

### Ex 1.2 — SQL Injection manual no DVWA
Acesse o DVWA (nível Low) → SQL Injection. Sem ferramentas, tente `' OR '1'='1` e observe.
_O que aprender:_ como a SQLi funciona no nível mais fundamental.

### Ex 1.3 — Capturar credenciais HTTP no Wireshark
Faça login no DVWA via HTTP (não HTTPS) e capture com o filtro `http.request.method == POST`.
_O que aprender:_ por que HTTP sem HTTPS expõe credenciais.

### Ex 1.4 — Primeiro Metasploit
```
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.200.10
run
# Você deve receber um shell! Digite: id, whoami, ls /
```
📖 [Metasploit Unleashed](https://www.offensive-security.com/metasploit-unleashed) · siga o curso em paralelo.

### Ex 1.5 — Primeiro hash crack com a RTX 4060
```bash
echo '5f4dcc3b5aa765d61d8327deb882cf99' > hash.txt
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt
# Resultado esperado: password
```
_O que aprender:_ como senhas MD5 sem salt são trivialmente quebráveis.

## 🟡 Nível 2 — Intermediário (meses 2–4)

**Meta:** encadear técnicas de forma independente e usar Red e Blue ao mesmo tempo.

### Ex 2.1 — Pentest completo documentado
Comprometa o Metasploitable3 Linux até root, **sem walkthroughs**. Ferramentas: Nmap, searchsploit, Metasploit, LinPEAS.
_Entregável:_ relatório no formato PTES — recon → exploração → pós-exploração → recomendações.

### Ex 2.2 — Purple Team: atacar e detectar
Repita qualquer ataque do Nível 1 e analise os logs no Wazuh. Em que horário exato o scan começou? Qual CVE foi usada? Qual usuário caiu?
_O que aprender:_ a perspectiva dual — como o Blue Team enxerga um ataque.

### Ex 2.3 — Escalada no Windows
Obtenha shell limitado no MSF3 Windows e escale para SYSTEM via WinPEAS.
📖 [Windows PrivEsc (TryHackMe)](https://tryhackme.com/room/windows10privesc)

### Ex 2.4 — Primeira máquina VulnHub
[Kioptrix Level 1](https://www.vulnhub.com/entry/kioptrix-level-1-1,22) — clássica de iniciante. Baixe, importe no VirtualBox e comprometa **sem walkthrough**.

## 🔴 Nível 3 — Avançado (meses 5+)

**Meta:** pensar como profissional, operar sem guias e documentar no padrão de mercado.

### Ex 3.1 — Comprometimento completo de AD
Enumere o AD, faça Kerberoasting, quebre com a RTX 4060 e escale até Domain Admin. Ferramentas: BloodHound, CrackMapExec, Impacket, Mimikatz.
_Entregável:_ relatório com todos os caminhos de ataque encontrados.

### Ex 3.2 — Purple Team avançado
Execute o ataque do Ex 3.1 monitorando o Wazuh em tempo real. Crie regras que detectem Kerberoasting, Pass-the-Hash e DCSync — e, do lado ofensivo, tente executar o ataque sem disparar as regras que você mesmo criou.

### Ex 3.3 — Análise de malware real
Baixe um sample do MalwareBazaar (ransomware ou RAT) na Flare-VM. Estática: strings, Detect-It-Easy, Ghidra. Dinâmica: Procmon + Regshot + Wireshark + x64dbg.
_Entregável:_ relatório de IOCs com mapeamento MITRE ATT&CK.

### Ex 3.4 — Participar de CTF
[CTFtime.org](https://ctftime.org) — calendário de CTFs. Comece pelos Jeopardy (categorias isoladas) antes de tentar Attack/Defense.

---

⏮️ Anterior: **[Parte 11 — Resposta a incidentes](11-resposta-a-incidentes.md)** · ⏭️ Próximo: **[Parte 13 — Desafios práticos](13-desafios-praticos.md)** · [↩︎ Índice](../README.md)
