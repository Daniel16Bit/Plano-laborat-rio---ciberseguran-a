# Parte 17 — Segurança e isolamento do laboratório

Esta é a parte que você **não** pode pular. Um lab de segurança mal isolado deixa de ser aprendizado e vira risco — para a sua própria rede e, no caso de malware, para muito além dela. As regras abaixo existem para que você pratique com liberdade total sem nunca colocar em perigo nada fora do ambiente controlado. Leia, entenda e siga sempre.

## 17.1 Regras de ouro

| # | Regra |
|---|---|
| 1 | **Nunca** coloque VMs de ataque em modo Bridge conectadas à sua rede doméstica real |
| 2 | **Nunca** execute malware real fora das VMs na MalNET — e nunca com internet ativa |
| 3 | Use snapshots **antes** de cada exercício — restaure **depois** |
| 4 | VMs da MalNET usam exclusivamente Internal Network — sem NAT, sem Bridge |
| 5 | Não use credenciais reais em VMs do lab (crie credenciais fictícias) |
| 6 | Use nomes de domínio internos (`corp.lab`, `empresa.local`) — nunca domínios reais |
| 7 | Jamais aplique técnicas do lab em sistemas que você não possui — **é crime** |
| 8 | Desabilite o compartilhamento de área de transferência com as VMs durante ataques |

## 17.2 Configurações de isolamento no VirtualBox

```bash
# Verificar que uma VM está na rede correta
VBoxManage showvminfo 'Metasploitable3-Linux' | grep 'NIC 1'
# Deve mostrar: NAT Network 'TargetNET' — nunca 'Bridged Adapter'

# Verificar que a MalNET é Internal
VBoxManage showvminfo 'REMnux' | grep 'NIC 1'
# Deve mostrar: Internal Network 'MalNET'
```

Desabilitar o compartilhamento de área de transferência:
`VM → Configurações → Geral → Avançado → Área de transferência: Desabilitar`

## 17.3 Verificar o isolamento de rede

```bash
# Dentro da VM de malware — confirmar que NÃO tem acesso à internet
ping 8.8.8.8
# Deve falhar (Request timeout). Se não falhar, há erro de configuração!

# Dentro de uma VM alvo — confirmar que não alcança a rede doméstica
ping 192.168.1.1   # IP do seu roteador
# Deve falhar — VMs da TargetNET não devem chegar à rede real
```

## 17.4 Rotina de laboratório

| Momento | Ação |
|---|---|
| Antes | Snapshot de todas as VMs que serão usadas |
| Durante | Documente cada passo — mantenha um diário de laboratório |
| Depois | Restaure as VMs ao snapshot "Estado Limpo" |
| Semanal | Revise os logs do Wazuh — o que passou despercebido? |
| Mensal | Atualize as ferramentas: `apt update` no Kali host |
| Mensal | Adicione uma nova VM ou técnica ao lab |

## ⚖️ Lei 12.737/2012 — Lei Carolina Dieckmann

No Brasil, acessar sistema informático alheio sem autorização é **crime**, com pena de 3 meses a 1 ano de reclusão + multa. Pratique **apenas** em sistemas que você mesmo criou ou para os quais tem autorização expressa por escrito. O laboratório existe justamente para você aprender tudo isso de forma legal e segura.

---

⏮️ Anterior: **[Parte 16 — TryHackMe](16-tryhackme.md)** · ⏭️ Próximo: **[Parte 18 — Glossário](18-glossario.md)** · [↩︎ Índice](../README.md)
