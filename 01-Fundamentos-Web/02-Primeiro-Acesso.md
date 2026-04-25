# A Dinâmica do 1º Acesso

## O que acontece quando você digita uma URL?

Parece simples, mas tem bastante coisa acontecendo por baixo dos panos. Aqui está o fluxo completo:

1. Você digita `https://www.exemplo.com` no navegador
2. O navegador monta a URL completa: `https://www.exemplo.com/`
3. O navegador inclui o cabeçalho `Host: www.exemplo.com` na requisição HTTP
4. O sistema faz uma consulta DNS para converter `www.exemplo.com` em um IP, tipo `192.168.1.1`
5. O navegador abre uma conexão TCP com esse IP na porta **443** (HTTPS) ou **80** (HTTP)
6. A requisição enviada tem o formato: `GET / HTTP/1.1`
7. O servidor processa e responde com código **200 OK** (ou outro código de status)
8. O navegador recebe a resposta e renderiza a página

---

## Autenticação via Cookie

Um fluxo típico de login funciona assim:

1. Usuário faz login - servidor valida e responde com `Set-Cookie: sessao=abc123`
2. Navegador armazena o cookie
3. Nas próximas requisições, o navegador envia automaticamente: `Cookie: sessao=abc123`
4. O servidor identifica o usuário pelo cookie

---

## Redirect (Redirecionamento)

Quando você acessa uma URL e o servidor quer te mandar para outro lugar:

```
GET / HTTP/1.1
→ 302 Found
→ Location: https://novo-endereco.com
```

O navegador então faz uma nova requisição para a URL indicada no `Location`.

---

## Arquitetura de uma aplicação web

Uma aplicação web simples tem basicamente três camadas:

```
[Usuário/Navegador]
       ↓  requisição HTTP
[Servidor Web] (Apache, Nginx, IIS)
       ↓
[Aplicação] (PHP, Python, Java, Node.js)
       ↓
[Banco de Dados]
```

Entender essa arquitetura é importante porque cada camada pode ter suas próprias vulnerabilidades.
