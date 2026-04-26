# Autenticação HTTP

## Basic Authentication

O método mais simples. O cliente codifica `usuário:senha` em Base64 e coloca no cabeçalho `Authorization`.

```
Authorization: Basic YWxhZGRpbjpvcGVuc2VzYW1l
```

Decodificando o valor acima em Base64: `aladdin:opensesame`

⚠️ **Problema**: a codificação em Base64 não é criptografada. Se a conexão não for HTTPS, qualquer pessoa na rede pode capturar e decodificar as credenciais em segundos.

---

## Digest Authentication

Mais seguro que o Basic. Em vez de enviar a senha diretamente, o servidor envia um **desafio** (nonce) e o cliente responde com um **hash** da senha combinada com esse desafio.

```
Authorization: Digest username="alice", realm="example.com",
               qop=auth, nonce="dcd98b7102dd2f0e8b11d0f600bfb0c093",
               uri="/resource", response="6629fae49393a05397450978507c4ef1"
```

- A senha nunca viaja pela rede
- Mas ainda é vulnerável a ataques de replay se mal implementado

---

## Bearer Token Authentication

Muito usado em APIs modernas. O cliente envia um **token de acesso** no cabeçalho, sem precisar reenviar usuário e senha a cada requisição.

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

- Frequentemente combinado com **JWT** (JSON Web Token)
- O token tem validade definida
- Se vazado, o atacante pode agir como o usuário até o token expirar

---

## Dica para pentest

Sempre verifique o cabeçalho `Authorization` nas requisições interceptadas. Se for Basic Auth, decodifique imediatamente em Base64. Se for Bearer, tente decodificar o JWT e veja o payload.

```bash
# Decodificando Basic Auth
echo "YWxhZGRpbjpvcGVuc2VzYW1l" | base64 -d

# Decodificando o payload de um JWT (segunda parte)
echo "eyJzdWIiOiIxMjM0NTY3ODkwIn0" | base64 -d
```
