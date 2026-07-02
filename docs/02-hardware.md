# Parte 2 — Hardware: análise e decisões do desktop

Hardware define o que o seu lab consegue (e o que não consegue) fazer. A boa notícia é que este desktop tem um trunfo raro em ambiente doméstico — uma GPU dedicada que transforma quebra de senha de "espera de horas" em "resultado instantâneo". A parte que exige disciplina é a RAM. Esta seção mostra como tirar o máximo de cada componente.

## 2.1 Análise completa do hardware

| Componente | Especificação | Avaliação para o lab |
|---|---|---|
| **CPU** | Ryzen 5 5600GT — 6C/12T, 4.6 GHz | Excelente. Roda 3–4 VMs simultâneas com folga |
| **GPU** | RTX 4060 8 GB — 3584 CUDA cores | 🏆 Diferencial enorme — Hashcat ~50x mais rápido que na CPU |
| **RAM** | 16 GB DDR4 | ⚠️ Principal limitante — exige gerenciamento (seção 2.3) |
| **Disco Linux** | 256 GB SSD | ⚠️ Apertado para VMs — resolvido com a partição Windows (seção 2.4) |
| **Disco Windows** | ~744 GB | ✅ Espaço para VMs pesadas e wordlists |
| **Wi-Fi** | Placa integrada | Para wireless, um adaptador Alfa USB é recomendado |
| **Ethernet** | Cabo disponível | Comunicação entre VMs e acesso à internet do host |
| **Dual Boot** | Kali + Windows 11 | Kali = Red Team · Windows 11 = ferramentas Blue Team nativas |

### Como usar o dual boot no lab

- 🔴 **Kali Linux (Red Team):** hypervisor (VirtualBox), VMs de ataque e Hashcat com a RTX 4060.
- 🔵 **Windows 11 (Blue Team):** Wazuh agent, Splunk Universal Forwarder, Sysinternals Suite, Event Viewer, Autopsy — ferramentas que rodam melhor no Windows nativo.

Na prática, você **boota no Kali** para atacar e **reinicia no Windows** para analisar os logs. Ou, melhor ainda: rode o Wazuh como serviço no Windows 11 — ele coleta os logs mesmo enquanto você está do lado do Kali.

## 2.2 RTX 4060 — Hashcat com GPU (a grande vantagem)

A RTX 4060 é o componente mais estratégico do lab. O Hashcat com CUDA é **50–100x mais rápido** que na CPU, o que torna os exercícios de quebra de senha realistas e imediatos.

| Tipo de hash | CPU Ryzen 5600GT | RTX 4060 CUDA | Ganho |
|---|---|---|---|
| MD5 | ~300 MH/s | ~15.000 MH/s | ~50x |
| NTLM (Windows) | ~500 MH/s | ~25.000 MH/s | ~50x |
| SHA-256 | ~120 MH/s | ~4.500 MH/s | ~37x |
| WPA/WPA2 | ~80 kH/s | ~900 kH/s | ~11x |
| bcrypt (custo 5) | ~3.000 H/s | ~30.000 H/s | ~10x |
| NetNTLMv2 | ~300 MH/s | ~14.000 MH/s | ~47x |

> ⚡ Com a RTX 4060, percorrer o `rockyou.txt` inteiro (14M de senhas) contra hashes NTLM leva **menos de 0,001 segundo**. Isso deixa os exercícios de Kerberoasting, Pass-the-Hash e hashdump extremamente didáticos — você vê o resultado na hora.

### 2.2.1 Instalar drivers NVIDIA e CUDA no Kali

> ⚠️ **O Hashcat roda no HOST Kali, não dentro das VMs.** As VMs não têm acesso direto à GPU. Extraia os hashes das VMs e traga para o Kali host para quebrar com a RTX 4060.

```bash
# 1. Verificar detecção da placa
lspci | grep -i nvidia
# Deve retornar: NVIDIA Corporation AD107 [GeForce RTX 4060]

# 2. Instalar drivers e CUDA
sudo apt update
sudo apt install -y nvidia-driver nvidia-cuda-toolkit
sudo reboot

# 3. Após reiniciar — verificar instalação
nvidia-smi
# Deve mostrar: RTX 4060 | CUDA Version: 12.x

# 4. Instalar Hashcat
sudo apt install -y hashcat

# 5. Confirmar GPU detectada
hashcat -I
# Deve listar: CUDA - NVIDIA GeForce RTX 4060

# 6. Benchmark para confirmar velocidade
hashcat -b -m 1000
# Esperado: ~20.000 a 28.000 MH/s em NTLM
```

- [Kali NVIDIA Docs](https://www.kali.org/docs/general-use/install-nvidia-drivers-on-kali-linux-in-vmware)
- [Hashcat CUDA FAQ](https://hashcat.net/wiki/doku.php?id=frequently_asked_questions)

### 2.2.2 Fluxo de trabalho — GPU cracking

```bash
# Passo 1: na VM comprometida, extraia os hashes (via Meterpreter)
hashdump
download /etc/shadow /tmp/shadow_extraido.txt

# Passo 2: no Kali HOST, quebre com a RTX 4060
hashcat -m 1000 ntlm_hashes.txt /usr/share/wordlists/rockyou.txt   # NTLM (Windows)
hashcat -m 1800 shadow.txt /usr/share/wordlists/rockyou.txt        # SHA-512 (/etc/shadow)

# Com regras (muito mais poderoso)
hashcat -m 1000 ntlm.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Kerberoasting TGS tickets
hashcat -m 13100 tgs_hashes.txt rockyou.txt

# Ver hashes já quebrados
hashcat -m 1000 ntlm.txt --show
```

- [OneRuleToRuleThemAll](https://github.com/NotSoSecure/password_cracking_rules) · regra única muito eficaz — baixe para o lab
- [Hashcat Rules Wiki](https://hashcat.net/wiki/doku.php?id=rule_based_attack)

## 2.3 Gerenciamento de RAM — 16 GB com inteligência

16 GB é o limite. A solução não é heroísmo: é **não ligar tudo de uma vez**. Cada tipo de exercício tem seu perfil de VMs ativas.

| Perfil de exercício | VMs ligadas | RAM total (aprox.) |
|---|---|---|
| Web Hacking | Kali VM + DVWA | ~3 GB |
| Pentest Linux | Kali VM + Metasploitable3 Linux | ~5 GB |
| Pentest Windows | Kali VM + Metasploitable3 Windows | ~6 GB |
| Active Directory | Kali VM + Windows Server AD | ~7 GB |
| Rede completa | Kali VM + pfSense + MSF3 Linux | ~7,5 GB |
| Blue Team | Wazuh Server VM + qualquer alvo | ~6 GB |
| Análise de malware | REMnux + Flare-VM (rede isolada) | ~6 GB |
| Lab completo (avançado) | Kali + MSF3 Linux + MSF3 Win + pfSense | ~11–13 GB ⚠️ |

### RAM mínima por VM (use no VirtualBox)

- **Kali VM (atacante):** 2 GB — XFCE é leve
- **Metasploitable 3 Linux:** 1–1,5 GB — sistema antigo
- **Metasploitable 3 Windows:** 2–3 GB — Win Server 2008 pede mais
- **DVWA (Ubuntu mínimo):** 512 MB–1 GB — só Apache + PHP
- **Windows Server 2019 AD:** 3–4 GB — AD DS + DNS + Kerberos
- **pfSense:** 512 MB — só roteia pacotes
- **REMnux:** 1,5–2 GB
- **Flare-VM (Win10 RE):** 3–4 GB

> **Regra:** reserve 2–3 GB para o Kali host. Sobram ~13–14 GB para as VMs.

> 💡 Se houver verba (~R$280), subir para **32 GB de RAM** elimina esse limitante e libera rodar o lab completo sem se preocupar com gerenciamento.

## 2.4 Armazenamento — 256 GB Linux + 744 GB Windows

A estratégia é híbrida: **VMs pesadas vivem na partição Windows** (744 GB), montada dentro do Kali. Resultado: ~400 GB para VMs, sem gastar nada.

- **VMs leves** (DVWA, pfSense, Kali VM) ficam na partição Linux.
- **VMs pesadas** (MSF3 Win, AD Server, Flare-VM) ficam na partição Windows.

```bash
# Descobrir o UUID da partição Windows
sudo blkid | grep ntfs

sudo mkdir -p /mnt/windows

# Montar manualmente primeiro (teste)
sudo mount -t ntfs-3g /dev/nvme0n1pX /mnt/windows

# Para montar automaticamente no boot, edite /etc/fstab e adicione:
# UUID=SEU-UUID /mnt/windows ntfs-3g defaults,uid=1000,gid=1000,umask=0022 0 0
sudo nano /etc/fstab

# Link simbólico para o diretório de VMs
ln -s /mnt/windows/VMs ~/VMs
# No VirtualBox, use ~/VMs/ como pasta padrão das máquinas
```

| Localização | Espaço | O que guardar |
|---|---|---|
| Linux `/` (sistema) | ~50 GB | Kali + `kali-linux-everything` + ferramentas |
| Linux `/VMs` | ~100 GB | VMs leves: Kali VM, DVWA, pfSense, REMnux |
| Linux `/wordlists` | ~30 GB | Rockyou, SecLists — acesso direto pelo Hashcat |
| Windows `D:\VMs` | ~300 GB | VMs pesadas: MSF3 Win, AD Server, Flare-VM |
| Windows `D:\Wordlists` | ~50 GB | CrackStation (15 GB), wordlists extras |
| Windows `D:\Captures` | ~50 GB | PCAPs do Wireshark, dumps de memória |

---

⏮️ Anterior: **[Parte 1 — Fundamentos](01-fundamentos.md)** · ⏭️ Próximo: **[Parte 3 — Arquitetura](03-arquitetura.md)** · [↩︎ Índice](../README.md)
