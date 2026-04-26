# Same-Origin Policy (SOP) e CORS

## Same-Origin Policy (SOP)

É uma das proteções mais importantes dos navegadores. A regra é simples:

> **JavaScript de um domínio só pode acessar conteúdo do mesmo domínio.**

### O que define "mesma origem"?

Três coisas precisam ser iguais:

| Componente | Exemplo |
|------------|---------|
| Protocolo | `https://` |
| Domínio | `exemplo.com` |
| Porta | `:443` |

Se qualquer um dos três for diferente, são origens diferentes e o SOP entra em ação.

### Exemplo prático

```
http://banco.com.br/conta   ← você está aqui, logado
http://malicioso.com         ← seu amigo te mandou acessar isso
```

Um JavaScript rodando em `malicioso.com` **não consegue** ler a resposta de uma requisição feita para `banco.com.br` o SOP bloqueia.

> ⚠️ **Importante**: o navegador ainda *faz* a requisição, mas não entrega a resposta ao JavaScript malicioso.

### O que o SOP não protege?

- Imagens, CSS e scripts JS de outras origens são carregados normalmente
- Formulários podem enviar dados para qualquer origem (CSRF explora isso)

### Exceções do SOP

- `window.location`
- `document.domain`
- Cross-window messaging (`postMessage`)
- CORS

---

## CORS (Cross-Origin Resource Sharing)

Quando uma aplicação legítima precisa fazer requisições cross-origin (ex: um frontend em `app.com` acessando uma API em `api.com`), o CORS entra para permitir isso de forma controlada.

### Como funciona?

O servidor diz ao navegador o que é permitido através de cabeçalhos HTTP específicos.

### Tipos de requisições CORS

**1. Requisição Simples** (navegador faz direto)
- Métodos: `GET`, `HEAD`, `POST`
- Content-Type: `application/x-www-form-urlencoded`, `multipart/form-data`, ou `text/plain`

**2. Requisição de Preflight** (navegador pergunta primeiro)
- Para métodos como `PUT`, `DELETE` ou `POST` com `application/json`
- O navegador manda uma requisição `OPTIONS` primeiro:

```
OPTIONS /recurso HTTP/1.1
Access-Control-Request-Method: DELETE
Access-Control-Request-Headers: Front-End-Https
```

O servidor responde dizendo o que aceita:
```
HTTP/1.1 200 OK
Access-Control-Allow-Methods: POST, GET, OPTIONS, DELETE
Access-Control-Allow-Headers: Front-End-Https
Access-Control-Max-Age: 3600
```

**3. Requisição com Credenciais**
- Cookies e headers de autenticação não são enviados cross-origin por padrão
- O desenvolvedor precisa usar `withCredentials: true` no XMLHttpRequest

### Cabeçalhos CORS

| Cabeçalho | Quem envia | O que faz |
|-----------|-----------|-----------|
| `Access-Control-Allow-Origin` | Servidor | Define quais origens são permitidas |
| `Access-Control-Allow-Methods` | Servidor | Métodos HTTP permitidos |
| `Access-Control-Allow-Headers` | Servidor | Headers permitidos na requisição |
| `Access-Control-Allow-Credentials` | Servidor | Permite envio de cookies |
| `Access-Control-Max-Age` | Servidor | Tempo de cache do preflight |
| `Origin` | Cliente | Informa a origem da requisição |

### Configuração perigosa

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

Isso nunca deveria estar junto. Permitir qualquer origem E credenciais é uma vulnerabilidade grave.

### Outro caso perigoso

Quando o servidor espelha cegamente a origem:
```python
# Código vulnerável
response.headers['Access-Control-Allow-Origin'] = request.headers['Origin']
```

Isso permite que qualquer origem acesse os recursos, incluindo origens maliciosas.
