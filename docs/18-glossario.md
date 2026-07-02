# Parte 18 — Glossário essencial

Uma página para consultar rápido sempre que um termo aparecer e você não lembrar o que significa.

| Termo | Definição |
|---|---|
| **APT** | Advanced Persistent Threat — atacante sofisticado com acesso prolongado |
| **Attack Surface** | Conjunto de todos os pontos de entrada possíveis para um atacante |
| **Brute Force** | Ataque que testa sistematicamente combinações de senha/chave |
| **Buffer Overflow** | Dados excedem o buffer alocado, sobrescrevendo memória adjacente |
| **C2 / Command & Control** | Infraestrutura para controlar sistemas comprometidos remotamente |
| **CVE** | Common Vulnerabilities and Exposures — identificador único de vulnerabilidade |
| **CVSS** | Score de 0 a 10 para a severidade de vulnerabilidades |
| **Domain Controller** | Servidor central do AD — autentica usuários e gerencia políticas |
| **Escalada de Privilégios** | Obter mais permissões do que as concedidas inicialmente |
| **Exploit** | Código/técnica que aproveita vulnerabilidade para ações não autorizadas |
| **Golden Ticket** | Ticket Kerberos forjado com o hash do krbtgt — acesso permanente ao AD |
| **Hash** | Função one-way de tamanho fixo — MD5, SHA-256, NTLM, bcrypt |
| **IOC** | Indicator of Compromise — evidência de comprometimento: IP, hash, domínio |
| **Kerberoasting** | Extrair TGS tickets de contas AD e quebrar offline |
| **Lateral Movement** | Mover-se para outros sistemas após o comprometimento inicial |
| **Meterpreter** | Payload avançado do Metasploit — na memória, difícil de detectar |
| **MITM** | Man-in-the-Middle — intercepta a comunicação entre duas partes |
| **NTLM Hash** | Hash de senha Windows — alvo de Pass-the-Hash e Kerberoasting |
| **Pass-the-Hash** | Usar o hash NTLM para autenticar sem conhecer a senha original |
| **Payload** | Código entregue pelo exploit para executar ações no alvo |
| **Persistence** | Manter acesso mesmo após reboot ou troca de senha |
| **Pivoting** | Usar um sistema comprometido como trampolim para atacar outros |
| **Reverse Shell** | Shell que a vítima abre de volta para o atacante — contorna firewall |
| **SIEM** | Centraliza logs e detecta ameaças — Wazuh, Splunk, Elastic |
| **SPN** | Service Principal Name — conta de serviço no AD, alvo do Kerberoasting |
| **TTP** | Tactics, Techniques, Procedures — metodologias do atacante (MITRE ATT&CK) |
| **Zero-Day** | Vulnerabilidade desconhecida pelo fabricante — sem patch disponível |

---

🛡️ **Laboratório personalizado para o seu desktop** — Ryzen 5600GT · RTX 4060 CUDA · 16 GB RAM · Dual Boot Kali + Windows 11

_"Hack the planet — ethically. Estude, pratique, respeite a lei."_

---

⏮️ Anterior: **[Parte 17 — Isolamento e segurança](17-isolamento-e-seguranca.md)** · [↩︎ Índice](../README.md)
