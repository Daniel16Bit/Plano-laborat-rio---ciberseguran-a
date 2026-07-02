# 🛡️ Laboratório Pessoal de Cibersegurança

Guia completo para montar, do zero, um laboratório de cibersegurança **ofensivo e defensivo** rodando inteiramente em um desktop doméstico. A ideia é simples: um só computador, um ambiente 100% isolado, e você aprendendo a atacar e a defender no mesmo lugar — o famoso **Purple Team solo**, que é a forma mais rápida de evoluir estudando sozinho.

Todo o material nasceu de um plano personalizado para um desktop específico (as especificações estão logo abaixo), mas a estrutura serve para qualquer máquina parecida.

> ⚠️ **Uso educacional exclusivo.** No Brasil, invadir sistema alheio sem autorização é crime — **Lei 12.737/2012**. Estude, pratique, respeite a lei.

---

## 🤖 Sobre este material

A documentação foi criada com auxílio de Inteligência Artificial para acelerar o processo de escrita e organização. O conteúdo técnico, as decisões de estudo e a prática são de responsabilidade do autor.

---

## 🖥️ O desktop de referência

| Componente | Especificação | Papel no lab |
|---|---|---|
| **CPU** | AMD Ryzen 5 5600GT — 6C/12T, 4.6 GHz | Roda 3–4 VMs ao mesmo tempo com folga |
| **GPU** | RTX 4060 8 GB — 3584 CUDA cores | 🏆 O diferencial: Hashcat ~50x mais rápido que na CPU |
| **RAM** | 16 GB DDR4 | O principal limitante — exige gerenciamento consciente |
| **Disco** | 256 GB Linux (Kali) + ~744 GB Windows 11 | Estratégia híbrida: VMs pesadas na partição Windows |
| **Rede** | Wi-Fi + Ethernet | Ethernet para comunicação entre VMs |
| **Boot** | Dual Boot Kali + Windows 11 | Kali = Red Team · Windows 11 = ferramentas Blue Team nativas |

---

## 📚 Índice da documentação

### Fundamentos e planejamento
- **[Parte 1 — Fundamentos e conceitos](docs/01-fundamentos.md)** · Tríade CIA, Red/Blue/Purple Team, PTES, MITRE ATT&CK, Kill Chain, redes, Linux e Windows para segurança
- **[Parte 2 — Hardware do desktop](docs/02-hardware.md)** · Análise do hardware, RTX 4060 no Hashcat, gerenciamento de RAM e armazenamento
- **[Parte 3 — Arquitetura do laboratório](docs/03-arquitetura.md)** · VirtualBox vs VMware, topologia de rede e configuração do zero
- **[Parte 4 — Máquinas virtuais](docs/04-maquinas-virtuais.md)** · Tabela de VMs, ordem de montagem e instalação de cada uma

### Red Team (ataque)
- **[Parte 5 — Ferramentas Red Team](docs/05-red-team.md)** · Recon, Nmap, enumeração, Metasploit, pós-exploração, cracking com GPU e Active Directory
- **[Parte 7 — Web Security (OWASP Top 10)](docs/07-web-security.md)** · Cada vulnerabilidade explorada na prática no DVWA
- **[Parte 8 — Engenharia reversa e malware](docs/08-engenharia-reversa-e-malware.md)** · Ghidra, x64dbg, análise estática e dinâmica
- **[Parte 9 — Criptografia](docs/09-criptografia.md)** · Fundamentos, vulnerabilidades comuns e ataques práticos

### Blue Team (defesa)
- **[Parte 6 — Ferramentas Blue Team](docs/06-blue-team.md)** · Wazuh (SIEM), Suricata, Wireshark, forense e threat hunting
- **[Parte 11 — Resposta a incidentes](docs/11-resposta-a-incidentes.md)** · NIST SP 800-61 aplicado ao lab

### Prática guiada
- **[Parte 10 — Cenários de ataque detalhados](docs/10-cenarios-de-ataque.md)** · Do web comprometido ao AD full compromise, passo a passo
- **[Parte 12 — Exercícios progressivos](docs/12-exercicios-progressivos.md)** · Do iniciante ao avançado
- **[Parte 13 — 10 desafios práticos (CorpLab S.A.)](docs/13-desafios-praticos.md)** · Um pentest completo em formato de missões

### Carreira e apoio
- **[Parte 14 — Plataformas, recursos e links](docs/14-recursos-e-links.md)**
- **[Parte 15 — Roadmap de certificações](docs/15-certificacoes.md)** · eJPT, PNPT, OSCP, Security+
- **[Parte 16 — TryHackMe vale a pena?](docs/16-tryhackme.md)**
- **[Parte 17 — Segurança e isolamento](docs/17-isolamento-e-seguranca.md)**
- **[Parte 18 — Glossário](docs/18-glossario.md)**

---

## 🚀 Começando rápido

1. Instale o VirtualBox e crie as redes virtuais — [Parte 3](docs/03-arquitetura.md)
2. Suba as primeiras VMs: Kali atacante + DVWA — [Parte 4](docs/04-maquinas-virtuais.md)
3. Faça o primeiro Nmap e a primeira SQL Injection — [Parte 12](docs/12-exercicios-progressivos.md)
4. Configure a RTX 4060 no Hashcat — [Parte 2](docs/02-hardware.md)
5. Suba o Wazuh e veja seus ataques do lado defensor — [Parte 6](docs/06-blue-team.md)

---

## 🗺️ Topologia

```
┌──────────────────────────────────────────────────────┐
│              DESKTOP — LABORATÓRIO COMPLETO           │
│                                                      │
│  KALI LINUX HOST                                     │
│   • VirtualBox + Hashcat com RTX 4060 (nativo)      │
│                        │                            │
│  ┌─────────────────────▼──────────────────────────┐ │
│  │  LabNET 192.168.100.0/24                        │ │
│  │  [Kali VM .100] [pfSense .1] [Wazuh .150]       │ │
│  └─────────────────────┬──────────────────────────┘ │
│                        │                            │
│  ┌─────────────────────▼──────────────────────────┐ │
│  │  TargetNET 192.168.200.0/24                     │ │
│  │  [MSF3 Linux .10] [MSF3 Win .20]                │ │
│  │  [DVWA .30] [AD Server .40]                     │ │
│  └────────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────────┐ │
│  │  MalNET — Internal (sem acesso externo)        │ │
│  │  [REMnux .10] [Flare-VM .20]                   │ │
│  └────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────┘
```

---

_"Hack the planet — ethically."_
