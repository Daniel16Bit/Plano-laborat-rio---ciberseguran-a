# Parte 3 — Planejamento e arquitetura do laboratório

Aqui o lab deixa de ser ideia e vira topologia concreta: qual hypervisor usar, como desenhar as redes para que atacante e alvos conversem sem nunca tocar a sua rede doméstica, e a sequência exata para sair do zero até um ambiente funcional.

## 3.1 Hypervisor — VirtualBox vs VMware

| Critério | VirtualBox 7.x | VMware Workstation Pro |
|---|---|---|
| Preço | 100% gratuito e open source | Gratuito para uso pessoal desde 2024 |
| Performance | Boa para lab | Superior — snapshots mais rápidos |
| Redes virtuais | NAT, Host-Only, Internal, Bridge | VMnet customizável — mais flexível |
| Compatibilidade | Linux, Win, macOS | Linux, Win (macOS = Fusion) |
| Snapshots | Sim — funciona bem | Sim — mais rápido e estável |
| Recomendação | ✅ **Comece aqui** | ✅ Migre se quiser mais performance |

- [VirtualBox Download (Linux)](https://www.virtualbox.org/wiki/Linux_Downloads) · instale no Kali
- [VirtualBox Extension Pack](https://www.virtualbox.org/wiki/Downloads) · obrigatório para USB 2.0/3.0 nas VMs
- [VMware Workstation Pro (gratuito)](https://www.vmware.com/products/workstation-pro.html)

## 3.2 Arquitetura de rede — topologia detalhada

```
┌──────────────────────────────────────────────────────────────┐
│                  DESKTOP — LABORATÓRIO COMPLETO                │
│                                                                │
│  KALI LINUX HOST (sistema nativo)                              │
│   • VirtualBox rodando todas as VMs                            │
│   • Hashcat com RTX 4060 (nativo, sem VM)                      │
│   • Ferramentas de Red Team direto no host                     │
│                          │ VirtualBox                          │
│  ┌───────────────────────▼──────────────────────────────────┐ │
│  │  LabNET — 192.168.100.0/24 (NAT Network)                 │ │
│  │  [Kali VM Atacante .100] [pfSense Firewall .1]            │ │
│  │  [Wazuh Server .150]                                      │ │
│  └───────────────────────┬──────────────────────────────────┘ │
│                          │ pfSense roteia e filtra             │
│  ┌───────────────────────▼──────────────────────────────────┐ │
│  │  TargetNET — 192.168.200.0/24 (NAT separada)             │ │
│  │  [Metasploitable3 Linux .10] [Metasploitable3 Win .20]   │ │
│  │  [DVWA / bWAPP .30] [Windows Server AD .40]              │ │
│  │  [Ubuntu Server .50]                                      │ │
│  └──────────────────────────────────────────────────────────┘ │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  MalNET — Internal Network (TOTALMENTE ISOLADA)          │ │
│  │  [REMnux .10] [Flare-VM .20] [Cuckoo Sandbox .30]        │ │
│  └──────────────────────────────────────────────────────────┘ │
│                                                                │
│  WINDOWS 11 (dual boot — Blue Team nativo):                    │
│   Wazuh Agent → Wazuh Server VM · Splunk UF · Sysinternals    │
│   Event Viewer · Autopsy · análise forense                    │
└──────────────────────────────────────────────────────────────┘
```

### Tipos de rede no VirtualBox — o que usar

| Tipo | Comunicação | Uso no lab |
|---|---|---|
| **NAT Network** | VMs entre si + saída via NAT | LabNET e TargetNET — atacante ↔ alvos |
| **Host-Only** | VM ↔ Host apenas | Acesso ao Wazuh Server sem expor à rede real |
| **Internal Network** | VMs isoladas — sem acesso externo | MalNET — malware em quarentena total |
| **Bridged** | VM na rede física real | ⚠️ **Nunca** com alvos ativos — só para baixar ferramentas |

## 3.3 Configuração passo a passo — do zero ao lab funcional

### Passo 1 — Instalar o VirtualBox no Kali

```bash
sudo apt update
sudo apt install -y virtualbox virtualbox-ext-pack
sudo usermod -aG vboxusers $USER
# Faça logout e login para aplicar o grupo
virtualbox &   # abrir a interface gráfica
```

### Passo 2 — Criar as redes virtuais

1. VirtualBox → **Arquivo → Gerenciador de Rede → aba Redes NAT → `+`**
2. Criar **LabNET**: CIDR `192.168.100.0/24` — desmarque DHCP automático
3. Criar **TargetNET**: CIDR `192.168.200.0/24` — desmarque DHCP
4. Criar **MalNET**: aba **Redes Internas** → nome `MalNET`
5. Criar adaptador **Host-Only**: aba **Adaptadores Host-Only → `+`** → `192.168.50.0/24`

### Passo 3 — Ativar virtualização na BIOS (se ainda não fez)

```bash
# Verificar se já está ativo
grep -E 'vmx|svm' /proc/cpuinfo | head -1
# Se retornar algo, já está ativo.
# Se vier vazio: reinicie, entre na BIOS (Del/F2) e ative 'AMD-V' ou 'SVM Mode'
```

### Passo 4 — Configurar IPs estáticos nas VMs

Cada VM deve ter IP fixo. Exemplo para a Kali VM atacante:

```bash
# Dentro da Kali VM
sudo nmcli con mod 'Wired connection 1' \
  ipv4.addresses '192.168.100.100/24' \
  ipv4.gateway '192.168.100.1' \
  ipv4.dns '8.8.8.8' \
  ipv4.method manual
sudo nmcli con up 'Wired connection 1'
ip addr show   # confirmar
```

### Passo 5 — Snapshots (faça ANTES de cada exercício)

```bash
# Criar snapshot
VBoxManage snapshot 'Metasploitable3-Linux' take 'Estado-Limpo' --live

# Restaurar snapshot
VBoxManage snapshot 'Metasploitable3-Linux' restore 'Estado-Limpo'

# Listar snapshots de uma VM
VBoxManage snapshot 'Metasploitable3-Linux' list
```

---

⏮️ Anterior: **[Parte 2 — Hardware](02-hardware.md)** · ⏭️ Próximo: **[Parte 4 — Máquinas virtuais](04-maquinas-virtuais.md)** · [↩︎ Índice](../README.md)
