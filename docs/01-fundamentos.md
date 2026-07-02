# Parte 1 — Fundamentos e conceitos de cibersegurança

Antes de sair ligando VMs e rodando exploits, vale firmar o terreno. Esta parte é a base conceitual do laboratório: os princípios que sustentam tudo, os times que existem numa operação de segurança e as metodologias que profissionais de verdade seguem. Se você absorver isso, todo o resto do guia faz muito mais sentido.

## 1.1 O que é cibersegurança — a tríade CIA

Cibersegurança é a disciplina de proteger sistemas, redes e dados contra ataques, danos ou acesso não autorizado. No seu laboratório, você vai estudar os dois lados ao mesmo tempo — ofensivo **e** defensivo. Esse é o modelo **Purple Team solo**, e é o mais eficaz para quem aprende sozinho.

Tudo gira em torno de três princípios:

| Princípio | O que garante | Exemplo prático |
|---|---|---|
| 🔐 **Confidencialidade** | Só quem tem autorização acessa a informação | Criptografia AES-256, controle de acesso |
| ✅ **Integridade** | Os dados não foram alterados sem permissão | Hashes SHA-256, assinaturas digitais |
| ⚡ **Disponibilidade** | O sistema está acessível quando é preciso | Anti-DDoS, backups, redundância |

## 1.2 Red Team, Blue Team e Purple Team

| Equipe | Papel | O que você vai praticar |
|---|---|---|
| 🔴 **Red Team** | Atacante — simula adversários reais | Pentest, exploração, evasão, engenharia social |
| 🔵 **Blue Team** | Defensor — detecta e responde | SIEM, IDS, forense, resposta a incidentes |
| 💜 **Purple Team** | Colaboração Red + Blue simultânea | Você ataca **e** monitora no mesmo exercício |

### Por que o seu setup é ideal para Purple Team

Com o dual boot Kali (Red) e Windows 11 (Blue) no mesmo desktop, dá para:

- Atacar via Kali Linux e, em seguida, analisar os logs no Windows 11 com Wazuh/Splunk.
- Manter dois terminais abertos: um atacando, outro monitorando o SIEM em tempo real.
- Reproduzir, na sua mesa, exatamente como um ambiente corporativo enxerga um incidente.

## 1.3 Metodologias fundamentais

### 1.3.1 PTES — Penetration Testing Execution Standard

São as 7 fases que todo pentest profissional segue. Use essa estrutura como espinha dorsal de cada exercício do lab:

1. **Pre-engagement** — definir escopo, objetivos e regras de engajamento
2. **Intelligence Gathering** — OSINT e reconhecimento
3. **Threat Modeling** — identificar ativos e vetores
4. **Vulnerability Analysis** — descobrir vulnerabilidades
5. **Exploitation** — explorar as vulnerabilidades
6. **Post-Exploitation** — persistência, coleta de credenciais, movimento lateral
7. **Reporting** — documentar tudo profissionalmente

📖 [PTES — padrão completo](http://www.pentest-standard.org/index.php/Main_Page) · leitura obrigatória

### 1.3.2 MITRE ATT&CK

O framework mais usado pela indústria para mapear **Táticas, Técnicas e Procedimentos (TTPs)** de atacantes. Cada técnica que você praticar no lab tem uma entrada correspondente no ATT&CK.

- [MITRE ATT&CK](https://attack.mitre.org) · explore cada tática e técnica
- [ATT&CK Navigator](https://mitre-attack.github.io/attack-navigator) · mapa visual para rastrear o que já praticou

### 1.3.3 Cyber Kill Chain — as 7 fases do ataque

| Fase | O que o atacante faz | Como o defensor interrompe |
|---|---|---|
| 1. Reconhecimento | Pesquisa pública, OSINT, scanning | Minimizar exposição de informações |
| 2. Weaponization | Cria payload/exploit | Threat intelligence, patch management |
| 3. Delivery | Envia via e-mail, web, USB | Filtragem de e-mail, endpoint protection |
| 4. Exploitation | Executa o exploit na vítima | Patching, hardening, EDR |
| 5. Installation | Instala backdoor/RAT | App whitelisting, HIDS |
| 6. C2 | Controla o sistema comprometido | Firewall, DNS filtering, IDS |
| 7. Objectives | Rouba dados, cifra, destrói | DLP, monitoramento, resposta a incidentes |

📖 [Cyber Kill Chain original (Lockheed Martin)](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html)

## 1.4 Fundamentos de rede

### 1.4.1 Modelo OSI — 7 camadas

| # | Nome | Protocolos / exemplo | Relevância para segurança |
|---|---|---|---|
| 7 | Aplicação | HTTP, DNS, FTP, SMTP | XSS, SQLi, DNS spoofing, phishing |
| 6 | Apresentação | SSL/TLS, encoding | SSL stripping, ataques a criptografia |
| 5 | Sessão | NetBIOS, RPC | Session hijacking |
| 4 | Transporte | TCP, UDP | SYN flood, port scanning |
| 3 | Rede | IP, ICMP, ARP | IP spoofing, MITM, routing attacks |
| 2 | Enlace | Ethernet, Wi-Fi, MAC | ARP poisoning, MAC spoofing |
| 1 | Física | Cabo, rádio, fibra | Wiretapping, jamming |

### 1.4.2 Portas e protocolos essenciais

| Porta | Protocolo | Relevância no lab |
|---|---|---|
| 22 | SSH | Alvo de brute force; acesso remoto seguro |
| 23 | Telnet | Sem criptografia — capture credenciais no Wireshark |
| 80/443 | HTTP/HTTPS | Web hacking: SQLi, XSS, IDOR, file upload |
| 139/445 | SMB | EternalBlue (MS17-010), Pass-the-Hash, compartilhamentos |
| 389 | LDAP | Active Directory — enumeração de usuários e grupos |
| 88 | Kerberos | Kerberoasting, AS-REP Roasting, Golden Ticket |
| 3389 | RDP | BlueKeep, brute force, credential stuffing |
| 53 | DNS | DNS poisoning, tunneling C2, exfiltração |
| 3306 | MySQL | Dump de banco via SQLi ou acesso direto com credencial fraca |

- [Professor Messer — networking gratuito](https://www.professormesser.com)
- [Subnet Calculator](https://www.subnet-calculator.com) · para planejar as sub-redes do lab

## 1.5 Linux para segurança — comandos essenciais

| Comando | Uso em cibersegurança |
|---|---|
| `find / -perm -u=s 2>/dev/null` | Binários SUID — vetores de escalada de privilégios |
| `cat /etc/crontab` | Tarefas agendadas — mecanismo de persistência |
| `ss -tulnp` / `netstat -tulnp` | Portas em escuta e processos — detectar backdoors |
| `grep -r 'password' /var/ /etc/` | Credenciais hardcoded em arquivos de config |
| `getcap -r / 2>/dev/null` | Linux capabilities — vetor de escalada menos óbvio |
| `ps aux \| grep -v grep` | Lista todos os processos — identificar suspeitos |
| `iptables -L -n -v` | Regras de firewall — entender bloqueios |
| `history \| grep -i pass` | Histórico de comandos — credenciais expostas |

- [OverTheWire Bandit](https://overthewire.org/wargames/bandit) · o melhor recurso gratuito para aprender Linux na prática
- [Explainshell](https://explainshell.com) · explica cada parte de qualquer comando

## 1.6 Windows para segurança

### 1.6.1 Estrutura interna crítica

- **SAM Database** — hashes de senhas locais em `%SystemRoot%\System32\config\SAM`
- **NTDS.dit** — banco do AD no Domain Controller; contém **todos** os hashes do domínio
- **LSASS (`lsass.exe`)** — processo de autenticação; alvo principal do Mimikatz
- **Registry Run Keys** — `HKCU/HKLM\...\Run`; mecanismo clássico de persistência
- **WMI** — usado para movimento lateral e persistência
- **AppLocker / WDAC** — controles de execução; aprenda a configurar e a contornar

### 1.6.2 Event IDs críticos para o Blue Team

| Event ID | O que significa | Por que é crítico |
|---|---|---|
| 4624 | Login bem-sucedido | Base para detectar logins anômalos |
| 4625 | Login falho | Vários seguidos = brute force em andamento |
| 4648 | Login com credenciais explícitas | Indica Pass-the-Hash / movimento lateral |
| 4688 | Novo processo criado | Detecta `powershell.exe`, `cmd.exe` suspeitos |
| 4698 | Tarefa agendada criada | Vetor de persistência — alerta imediato |
| 4768/4769 | Kerberos TGT/TGS solicitado | Kerberoasting gera muitos 4769 rápidos |
| 7045 | Novo serviço instalado | Malware costuma instalar serviços |
| 1102 | Log de auditoria limpo | Atacante apagando rastros — alerta crítico |

- [SANS Windows Event Log Poster](https://www.sans.org/posters/windows-event-log-analysis)
- [Microsoft Security Auditing](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/security-auditing-overview)

---

⏭️ Próximo: **[Parte 2 — Hardware do desktop](02-hardware.md)** · [↩︎ Voltar ao índice](../README.md)
