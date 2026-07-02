# Parte 4 — Máquinas virtuais

As VMs são o coração do lab: são elas que você vai atacar, defender e observar. Esta parte traz a alocação de recursos pensada para 16 GB de RAM, a ordem inteligente de montagem (você não precisa subir tudo na primeira semana) e o passo a passo de instalação de cada máquina.

## 4.1 Tabela de VMs e alocação de recursos

| VM / Sistema | RAM | Disco | Rede | Finalidade |
|---|---|---|---|---|
| Kali VM (atacante) | 2 GB | 50 GB | LabNET | Ambiente de ataque isolado — Red Team |
| pfSense 2.7 | 512 MB | 8 GB | LabNET + Target | Firewall/roteador entre redes |
| Wazuh Server | 2 GB | 40 GB | LabNET + HO | SIEM central — Blue Team |
| Metasploitable3 Linux | 1,5 GB | 25 GB | TargetNET | Alvo Linux cheio de CVEs |
| Metasploitable3 Windows | 2,5 GB | 50 GB | TargetNET | Alvo Windows — MS17-010, RDP, IIS |
| DVWA + bWAPP | 1 GB | 15 GB | TargetNET | Web hacking: SQLi, XSS, IDOR |
| Windows Server 2019 AD | 3,5 GB | 55 GB | TargetNET | Active Directory completo |
| REMnux | 2 GB | 30 GB | MalNET | Análise de malware — isolada |
| Flare-VM (Win10) | 3,5 GB | 70 GB | MalNET | Engenharia reversa Windows |

### Armazenamento por VM

- **Partição Linux (~100 GB para VMs):** Kali VM, pfSense, Wazuh Server, DVWA
- **Partição Windows `D:\VMs` (~300 GB):** MSF3 Linux, MSF3 Windows, AD Server, REMnux, Flare-VM

Isso distribui a carga e resolve o espaço sem custo nenhum.

## 4.2 Ordem de montagem — semana a semana

Não tente subir o lab inteiro de uma vez. Cresça junto com a sua curva de aprendizado:

| Período | Adicionar | Objetivo |
|---|---|---|
| Semana 1 | Kali VM + DVWA | Primeiros ataques web — SQLi, XSS, file upload |
| Semana 2 | Metasploitable3 Linux | Exploração Linux — Metasploit, escalada de privilégios |
| Semana 3 | Wazuh Server VM | Começa o Blue Team — logs e alertas |
| Semana 4 | Metasploitable3 Windows | Ataques Windows — SMB, RDP, credenciais |
| Mês 2 | pfSense + AD Server | Simulação corporativa completa com domínio |
| Mês 3–4 | REMnux + Flare-VM | Análise de malware e engenharia reversa |

## 4.3 Download e instalação de cada VM

### 4.3.1 Metasploitable 3 — alvo principal

Máquina intencionalmente vulnerável da Rapid7 — base de quase todos os seus exercícios.

```bash
# Método Vagrant (oficial)
sudo apt install -y vagrant
git clone https://github.com/rapid7/metasploitable3
cd metasploitable3

vagrant up ubuntu1404   # build da versão Linux (30-60 min no 1º build)
vagrant up win2k8       # build da versão Windows

# Credenciais padrão Linux:   vagrant/vagrant
# Credenciais padrão Windows: Administrator/vagrant
```

📖 [Metasploitable3 no GitHub](https://github.com/rapid7/metasploitable3) · repositório oficial + wiki de vulnerabilidades

### 4.3.2 DVWA — alvo web

```bash
# Via Docker (mais rápido)
sudo apt install -y docker.io
sudo systemctl enable --now docker
sudo docker run -d -p 8080:80 --name dvwa vulnerables/web-dvwa

# Acesse:  http://127.0.0.1:8080
# Setup:   http://127.0.0.1:8080/setup.php  → Create/Reset Database
# Login padrão: admin / password
```

📖 [DVWA no GitHub](https://github.com/digininja/DVWA)

### 4.3.3 Wazuh Server — SIEM do lab

```bash
# Instalar via script oficial em VM Ubuntu 22.04
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash wazuh-install.sh -a
# Acesse a interface web: https://[IP-VM-WAZUH]

# Instalar o agente no Kali host
sudo apt install -y wazuh-agent
sudo nano /var/ossec/etc/ossec.conf
# <address>192.168.100.150</address>   <- IP do Wazuh Server VM
sudo systemctl enable --now wazuh-agent
```

- [Wazuh Docs](https://documentation.wazuh.com/current/index.html)
- [Wazuh Installation Guide](https://documentation.wazuh.com/current/installation-guide/index.html)

### 4.3.4 Windows Server 2019 + Active Directory

Trial gratuito de 180 dias da Microsoft — mais que suficiente para o lab.

1. Baixe em [microsoft.com/evalcenter](https://www.microsoft.com/evalcenter) → Windows Server 2019
2. Instale como VM com 3,5 GB de RAM na TargetNET com IP `192.168.200.40`
3. Depois de instalar: **Server Manager → Add Roles → Active Directory Domain Services**
4. Promova a controlador de domínio: domain name `corp.lab` — **anote a senha do Administrator**
5. Configure o DNS apontando para ele mesmo: `192.168.200.40`
6. Crie usuários vulneráveis para os exercícios de Kerberoasting

- [AD DS Setup Guide (Microsoft)](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/install-active-directory-domain-services--level-100-)
- [GOAD — Game of Active Directory](https://github.com/Orange-Cyberdefense/GOAD) · lab AD avançado — para quando tiver experiência

### 4.3.5 REMnux e Flare-VM — malware e RE

- [REMnux Download](https://remnux.org) · VM Linux para análise de malware
- [Flare-VM no GitHub](https://github.com/mandiant/flare-vm) · instale em VM Win10 — script PowerShell automático

> ⚠️ **MalNET totalmente isolada.** REMnux e Flare-VM ficam **exclusivamente** na rede Internal `MalNET`. Jamais conecte VMs de análise de malware a redes com acesso externo.

---

⏮️ Anterior: **[Parte 3 — Arquitetura](03-arquitetura.md)** · ⏭️ Próximo: **[Parte 5 — Red Team](05-red-team.md)** · [↩︎ Índice](../README.md)
