# 🛡️ Laboratório Pessoal de Cibersegurança

Guia completo para montar, do zero, um laboratório de cibersegurança **ofensivo e defensivo** rodando inteiramente em um desktop doméstico. A ideia é simples: um só computador, um ambiente 100% isolado, e você aprendendo a atacar e a defender no mesmo lugar — o famoso **Purple Team solo**, que é a forma mais rápida de evoluir estudando sozinho.

Todo o material nasceu de um plano personalizado para um desktop específico (as especificações estão logo abaixo), mas a estrutura serve para qualquer máquina parecida. Se o seu hardware for diferente, os números de RAM e disco mudam, mas o raciocínio continua o mesmo.

> ⚠️ **Uso educacional exclusivo.** Tudo aqui é para ser praticado em máquinas que você mesmo criou, dentro de um ambiente fechado. No Brasil, invadir sistema alheio sem autorização é crime — **Lei 12.737/2012 (Lei Carolina Dieckmann)**. Estude, pratique, respeite a lei. Os detalhes estão na [Parte 17](docs/17-isolamento-e-seguranca.md).

---

## 🖥️ O desktop de referência

| Componente | Especificação | Papel no lab |
|---|---|---|
| **CPU** | AMD Ryzen 5 5600GT — 6C/12T, 4.6 GHz | Roda 3–4 VMs ao mesmo tempo com folga |
| **GPU** | RTX 4060 8 GB — 3584 CUDA cores | 🏆 O diferencial: Hashcat ~50x mais rápido que na CPU |
| **RAM** | 16 GB DDR4 | O principal limitante — exige gerenciamento consciente |
| **Disco** | 256 GB Linux (Kali) + ~744 GB Windows 11 | Estratégia híbrida: VMs pesadas na partição Windows |
| **Rede** | Wi-Fi + Ethernet | Ethernet para comunicação entre VMs e acesso à internet |
| **Boot** | Dual Boot Kali + Windows 11 | Kali = Red Team · Windows 11 = ferramentas Blue Team nativas |

A jogada central é usar o **dual boot a seu favor**: você ataca a partir do Kali e analisa os logs no Windows 11 com Wazuh, Sysinternals e companhia. Dois lados da mesma moeda, na mesma máquina.

---

## 📚 Índice da documentação

Cada parte do guia virou um arquivo próprio dentro de [`docs/`](docs/), para você navegar, buscar e linkar direto o que precisa.

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
- **[Parte 14 — Plataformas, recursos e links](docs/14-recursos-e-links.md)** · Onde estudar, canais, livros e bookmarks obrigatórios
- **[Parte 15 — Roadmap de certificações](docs/15-certificacoes.md)** · eJPT, PNPT, OSCP, Security+ e o caminho entre elas
- **[Parte 16 — TryHackMe vale a pena?](docs/16-tryhackme.md)** · Análise honesta de custo x benefício
- **[Parte 17 — Segurança e isolamento](docs/17-isolamento-e-seguranca.md)** · As regras de ouro para não se machucar (nem infringir a lei)
- **[Parte 18 — Glossário](docs/18-glossario.md)** · Os termos essenciais em uma página

---

## 🚀 Começando rápido

Se você quer sair do papel hoje, esta é a sequência mínima:

1. **Instale o VirtualBox no Kali** e crie as redes virtuais isoladas — [Parte 3](docs/03-arquitetura.md).
2. **Suba as duas primeiras VMs**: Kali atacante + DVWA. É o suficiente para os primeiros ataques web — [Parte 4](docs/04-maquinas-virtuais.md).
3. **Faça o primeiro Nmap e a primeira SQL Injection** — [Parte 12](docs/12-exercicios-progressivos.md), Nível 1.
4. **Configure a RTX 4060 no Hashcat** para quebrar seu primeiro hash em segundos — [Parte 2](docs/02-hardware.md).
5. **Suba o Wazuh** e comece a enxergar os seus próprios ataques do lado defensor — [Parte 6](docs/06-blue-team.md).

A ordem completa, semana a semana, está na [Parte 4](docs/04-maquinas-virtuais.md#42-ordem-de-montagem--semana-a-semana).

---

## 🗺️ Topologia do laboratório

```
┌──────────────────────────────────────────────────────────────┐
│                  DESKTOP — LABORATÓRIO COMPLETO                │
│                                                                │
│  KALI LINUX HOST (nativo)                                      │
│   • VirtualBox rodando todas as VMs                            │
│   • Hashcat com RTX 4060 (nativo, sem VM)                      │
│   • Ferramentas Red Team direto no host                        │
│                          │ VirtualBox                          │
│  ┌───────────────────────▼──────────────────────────────────┐ │
│  │  LabNET — 192.168.100.0/24 (NAT)                          │ │
│  │  [Kali VM .100]  [pfSense .1]  [Wazuh Server .150]        │ │
│  └───────────────────────┬──────────────────────────────────┘ │
│                          │ pfSense roteia e filtra             │
│  ┌───────────────────────▼──────────────────────────────────┐ │
│  │  TargetNET — 192.168.200.0/24 (NAT separada)             │ │
│  │  [MSF3 Linux .10] [MSF3 Win .20] [DVWA .30]              │ │
│  │  [Windows Server AD .40] [Ubuntu .50]                     │ │
│  └──────────────────────────────────────────────────────────┘ │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  MalNET — Internal Network (TOTALMENTE ISOLADA)          │ │
│  │  [REMnux .10] [Flare-VM .20] [Cuckoo .30]                │ │
│  └──────────────────────────────────────────────────────────┘ │
│                                                                │
│  WINDOWS 11 (dual boot — Blue Team nativo):                    │
│   Wazuh Agent → Wazuh Server · Sysinternals · Autopsy         │
└──────────────────────────────────────────────────────────────┘
```

Detalhes de cada rede e o porquê de cada escolha estão na [Parte 3](docs/03-arquitetura.md).

---

## 📄 Guia original em PDF

O guia completo em PDF, formatado para leitura corrida, está no repositório:
**[`Laboratorio_Ciberseguranca_Desktop_v3.pdf`](Laboratorio_Ciberseguranca_Desktop_v3.pdf)**

A documentação em Markdown desta pasta é a versão navegável e sempre atualizada do mesmo conteúdo.

---

## ⚖️ Aviso legal

Este material é **exclusivamente educacional** e voltado para prática em ambiente isolado e controlado. Aplicar qualquer técnica aqui descrita contra sistemas de terceiros sem autorização expressa e por escrito é crime. A responsabilidade pelo uso é inteiramente de quem pratica.

_"Hack the planet — ethically. Estude, pratique, respeite a lei."_
