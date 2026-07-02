# Parte 9 — Criptografia: fundamentos e ataques

Criptografia é uma faca de dois gumes: bem usada, é a base da confidencialidade e da integridade; mal usada, vira o elo mais fraco. Esta parte cobre os fundamentos que você precisa dominar e, principalmente, os erros clássicos que tornam a criptografia quebrável — porque é isso que você vai explorar em CTFs e em análise de malware.

## 9.1 Fundamentos

| Tipo | Como funciona | Relevância para segurança |
|---|---|---|
| **Simétrica** | Mesma chave para cifrar e decifrar | AES-256 — criptografia de dados em massa |
| **Assimétrica** | Par chave pública/privada | RSA/ECC — TLS, SSH, assinatura digital |
| **Hash** | One-way, não reversível | SHA-256 para integridade; MD5 para senhas é **inseguro** |
| **HMAC** | Hash com chave — autenticação | Verifica autenticidade **e** integridade |

### Vulnerabilidades criptográficas comuns

- **MD5/SHA1 para senhas sem salt:** quebrável em segundos com Hashcat + rockyou.
- **RSA com módulo pequeno (512/1024 bits):** fatorável com o RsaCtfTool.
- **AES em modo ECB:** padrões visíveis no ciphertext — o clássico "penguin ECB".
- **Reuso de nonce em stream ciphers:** o XOR de dois ciphertexts revela o XOR dos plaintexts.
- **Padding Oracle Attack:** usa as mensagens de erro para decifrar dados AES-CBC.

## 9.2 Ferramentas e ataques práticos

### CyberChef — o canivete suíço da criptografia

Criado pelo GCHQ (inteligência britânica). Use para identificar e decodificar praticamente qualquer encoding. Operações mais úteis em CTFs e análise de malware:

- **From Base64** → de texto codificado para original
- **From Hex** → de hex para bytes
- **ROT13** → cifra clássica simples
- **XOR** → operação com chave
- **Magic** → detecta automaticamente o encoding (muito útil!)
- **MD5 / SHA256** → calcula hashes
- **AES Decrypt** → decifra com chave e IV conhecidos

🔗 [CyberChef online](https://gchq.github.io/CyberChef) · use no browser, sem instalação

### Identificar e quebrar hashes

```bash
hashid '5f4dcc3b5aa765d61d8327deb882cf99'
hash-identifier
```

Referência rápida de formatos:

```
MD5:    32 chars hex
SHA1:   40 chars hex
SHA256: 64 chars hex
NTLM:   32 chars hex (parece MD5, mas é diferente)
bcrypt: começa com $2b$ ou $2y$
```

### RSA fraco e OpenSSL

```bash
# RSA fraco — para CTFs
python3 RsaCtfTool.py --publickey pub.pem --private

# OpenSSL — manipular certificados
openssl x509 -in cert.pem -text -noout
openssl req -newkey rsa:2048 -x509 -days 365 -out cert.pem -keyout key.pem
```

## Recursos

- [CryptoHack](https://cryptohack.org) · plataforma gamificada de criptografia — excelente
- [Cryptopals Challenges](https://cryptopals.com) · desafios do básico ao avançado
- [Dan Boneh — Stanford Crypto](https://www.coursera.org/learn/crypto) · curso universitário gratuito

---

⏮️ Anterior: **[Parte 8 — Engenharia reversa](08-engenharia-reversa-e-malware.md)** · ⏭️ Próximo: **[Parte 10 — Cenários de ataque](10-cenarios-de-ataque.md)** · [↩︎ Índice](../README.md)
