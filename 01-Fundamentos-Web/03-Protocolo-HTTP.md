# Protocolo HTTP

## O que é o HTTP?

HTTP é o protocolo de comunicação da web. É um protocolo em **texto puro**, **stateless** (sem estado) e segue um modelo simples de requisição e resposta.

- **Texto puro**: dá pra ler e manipular facilmente, ótimo para pentest
- **Stateless**: o servidor não guarda o estado entre requisições, cada uma é independente

---

## Evolução do protocolo

### HTTP 1.0 (RFC 1945)
- Versão inicial
- Cada requisição abre e fecha uma nova conexão TCP

### HTTP 1.1 (RFC 2616)
- Adicionou `keep-alive` para reusar a conexão
- Novos métodos HTTP: `PUT`, `DELETE`, `OPTIONS`, `TRACE`, `CONNECT`

### HTTP 2
- **Protocolo binário** (não é mais texto puro)
- **Multiplexado**: várias requisições na mesma conexão simultaneamente
- **Compressão de cabeçalhos**: menor overhead
- **Server Push**: o servidor pode enviar recursos antes de o cliente pedir

> Como verificar se um servidor usa HTTP/2:
> ```bash
> curl --verbose --http2-prior-knowledge https://alvo.com
> nghttp -ans https://alvo.com
> ```

---

## Métodos HTTP

| Método | Uso |
|--------|-----|
| `GET` | Buscar recursos |
| `POST` | Enviar dados (criar) |
| `PUT` | Atualizar recurso completo |
| `DELETE` | Remover recurso |
| `PATCH` | Atualizar recurso parcialmente |
| `OPTIONS` | Listar métodos disponíveis |
| `HEAD` | Igual ao GET mas sem o body da resposta |
| `TRACE` | Retorna a própria requisição (diagnóstico) |
| `CONNECT` | Cria um túnel (usado em proxies) |

---

## Estrutura de uma Requisição HTTP

```
GET /pagina HTTP/1.1
Host: www.exemplo.com
User-Agent: Mozilla/5.0
Accept: text/html
Accept-Encoding: gzip, deflate
Connection: keep-alive

[body — só em POST, PUT, PATCH]
```

---

## Estrutura de uma Resposta HTTP

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 583
Cache-Control: max-age=3600

[body com o conteúdo]
```

---

## Códigos de Status HTTP

| Faixa | Significado | Exemplos |
|-------|-------------|---------|
| 2xx | Sucesso | 200 OK, 201 Created, 204 No Content |
| 3xx | Redirecionamento | 301 Moved, 302 Found, 304 Not Modified |
| 4xx | Erro do cliente | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found |
| 5xx | Erro do servidor | 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable |

> No pentest, erros 4xx e 5xx podem revelar informações sobre a aplicação. Um 500 inesperado pode indicar uma falha explorável.

---

## Por que HTTP 2 complica o pentest?

Com HTTP/2 sendo binário, o `telnet` para testar manualmente não funciona mais. Ferramentas como o **Burp Suite** precisam ter suporte explícito para HTTP/2 para interceptar corretamente.
