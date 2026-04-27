# Analisando o Frontend com DevTools

## O que é o DevTools?

Toda ferramenta de desenvolvedor embutida nos navegadores modernos. Essencial para entender como uma aplicação web funciona antes de testá-la.

**Como abrir:**
- Elementos: `Ctrl+Shift+C` (Windows/Linux) ou `Cmd+Option+C` (Mac)
- Console: `Ctrl+Shift+J` (Windows/Linux) ou `Cmd+Option+J` (Mac)
- Ou simplesmente: F12

---

## Painéis principais

| Painel | O que inspecionar |
|--------|------------------|
| **Elements** | HTML e CSS da página em tempo real |
| **Console** | Executar JavaScript, ver logs e erros |
| **Sources** | Código-fonte JS, adicionar breakpoints |
| **Network** | Todas as requisições feitas pela página |
| **Application** | LocalStorage, SessionStorage, Cookies, IndexedDB |
| **Security** | Certificados e problemas de segurança |
| **Performance** | Profiling de desempenho |
| **Lighthouse** | Auditoria de qualidade |

---

## Network - o mais útil para pentest

A aba **Network** mostra todas as requisições feitas pela página enquanto você navega. É possível:
- Ver endpoints de API que o frontend consome
- Inspecionar headers de requisição e resposta
- Copiar a requisição como cURL para usar no terminal ou no Burp
- Identificar parâmetros enviados

> Para mapear todos os endpoints de uma API, navegue pela aplicação com a aba Network aberta e filtre por `XHR` ou `Fetch`.

---

## Application - onde ficam os dados do cliente

Nessa aba você vê:
- **Cookies** - com todos os atributos (HttpOnly, Secure, SameSite, valor)
- **LocalStorage** - dados persistentes
- **SessionStorage** - dados de sessão
- **IndexedDB** - banco de dados do navegador

Qualquer dado sensível armazenado aqui (tokens, IDs, chaves) é um achado importante.

---

## Sources e Breakpoints

Útil para depurar e entender a lógica do JavaScript antes de testar.

### Como adicionar um breakpoint
1. Abra a aba **Sources**
2. No painel esquerdo, navegue até o arquivo `.js`
3. Clique com o botão direito na linha e selecione "Add breakpoint"
4. Execute a ação na página - o código pausa naquele ponto
5. Inspecione as variáveis no painel direito

### Tipos de breakpoints
- **Line-of-code**: pausa em uma linha específica
- **DOM change**: pausa quando um elemento HTML muda
- **XHR/Fetch**: pausa quando uma requisição é feita para uma URL específica
- **Event listener**: pausa quando um evento acontece (clique, submit, etc.)

---

## Blackboxing

Permite ignorar scripts de terceiros durante a depuração (como jQuery, Analytics). Assim você se foca só no código da aplicação.

Como usar:
1. Vá em **Sources**
2. Clique com botão direito no script
3. Selecione "Add script to ignore list"

---

## Console - executando JavaScript

Você pode executar JavaScript diretamente no console para:
- Testar payloads de XSS sem modificar o código
- Acessar objetos e variáveis do escopo global
- Verificar o que está no localStorage: `JSON.stringify(localStorage)`
- Testar se `document.cookie` está acessível (se sim, HttpOnly não está configurado)

```javascript
// Verificar cookies acessíveis
document.cookie

// Ver todo o localStorage
Object.keys(localStorage).forEach(k => console.log(k, localStorage.getItem(k)))

// Testar XSS no console (sem impacto real)
document.title = "XSS test"
```

---

## Snippets

A aba **Sources → Snippets** permite salvar scripts que você usa com frequência para reutilizar em diferentes sites durante o pentest.
