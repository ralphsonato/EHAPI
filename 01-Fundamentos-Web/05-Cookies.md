# Cookies

## O que são?

Cookies são pequenos arquivos de dados que o servidor manda pro navegador, que os guarda e envia de volta a cada requisição para o mesmo domínio. Foram criados justamente para contornar o fato do HTTP ser stateless, ou seja, para o servidor conseguir "lembrar" quem é você.

**Uso típico:** manter sessão de login, preferências do usuário, carrinho de compras.

---

## Como funcionam?

1. Você faz login
2. O servidor responde com: `Set-Cookie: sessao=abc123; HttpOnly; Secure`
3. Seu navegador armazena o cookie
4. Nas próximas requisições para aquele domínio, o navegador envia: `Cookie: sessao=abc123`

---

## Tipos de Cookies

| Tipo | Descrição |
|------|-----------|
| **Sessão** | Temporário — some quando o navegador fecha |
| **Persistente** | Tem data de expiração definida — fica mesmo fechando o navegador |
| **Terceiros** | Criado por domínios que não são o que você está visitando (muito usado para rastreamento) |

---

## Estrutura de um Cookie

Um cookie tem vários atributos:

```
Set-Cookie: nome=valor; Domain=exemplo.com; Path=/; Expires=Wed, 21 Oct 2025 07:28:00 GMT; Secure; HttpOnly; SameSite=Strict
```

| Atributo | O que faz |
|----------|-----------|
| `nome=valor` | A informação guardada |
| `Domain` | Para quais domínios o cookie é enviado |
| `Path` | Para quais caminhos dentro do domínio |
| `Expires` / `Max-Age` | Quando expira |
| `Secure` | Só enviado em conexões HTTPS |
| `HttpOnly` | JavaScript não consegue acessar — protege de XSS |
| `SameSite` | Controla envio em requisições cross-site (proteção contra CSRF) |

---

## Perspectiva de pentest

- **Sem `HttpOnly`**: o cookie pode ser roubado via XSS com `document.cookie`
- **Sem `Secure`**: o cookie pode vazar em conexões HTTP simples
- **Sem `SameSite`**: mais vulnerável a ataques CSRF
- **Valor previsível**: se o cookie de sessão for gerado de forma fraca, pode ser forjado

Sempre inspecione os cookies da aplicação testada. Um cookie de sessão com valor simples (tipo um número sequencial) é um sinal de alerta.
