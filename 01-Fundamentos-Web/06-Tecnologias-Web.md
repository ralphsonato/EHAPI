# Tecnologias Web

## Camadas da web

A web é construída em cima de três camadas principais:

| Tecnologia | Papel |
|------------|-------|
| **HTML** | Estrutura - o esqueleto da página |
| **CSS** | Estilo - aparência e layout |
| **JavaScript** | Comportamento - interatividade e dinamismo |

---

## HTML

Linguagem de marcação que estrutura o conteúdo. Usa tags para definir elementos.

```html
<!DOCTYPE html>
<html>
<head>
  <title>Exemplo</title>
</head>
<body>
  <h1>Título</h1>
  <p>Parágrafo com <a href="/link">um link</a></p>
</body>
</html>
```

Durante um pentest, os atributos das tags (como `action`, `href`, `src`, `value`) são pontos interessantes para investigar.

---

## CSS

Define a aparência visual dos elementos HTML. Pode ser inline, interno ou externo.

---

## JavaScript - A linguagem da web

JS é o que torna as páginas dinâmicas. Ele roda no navegador (client-side) e pode:

- Fazer requisições para o servidor sem recarregar a página (Ajax, Fetch API)
- Manipular o DOM em tempo real
- Armazenar dados localmente (localStorage, sessionStorage)
- Validar formulários antes de enviar

> ⚠️ Validações feitas **apenas no JavaScript do lado do cliente** são bypassáveis. Sempre dá pra interceptar e modificar a requisição com o Burp Suite antes de chegar no servidor.

---

## Linguagens Server-Side

Rodam no servidor e processam a lógica da aplicação. As mais comuns:

- **PHP** - ainda muito presente em aplicações legadas
- **Python** (Django, Flask, FastAPI)
- **Java** (Spring)
- **Node.js** (Express)
- **Ruby** (Rails)

---

## Frameworks Client-Side

Bibliotecas e frameworks JavaScript que facilitam a criação de interfaces:

- **React** - o mais popular atualmente
- **Angular** - muito usado em corporativos
- **Vue.js** - mais leve e simples
- **Ember** - menos comum

---

## Servidores Web

São eles que recebem as requisições HTTP e entregam as respostas:

| Servidor | Market share | Observação |
|----------|-------------|------------|
| **Apache** | 49% | Mais comum, gratuito |
| **Nginx** | 34% | Alta performance, gratuito |
| **Microsoft IIS** | 11% | Pago, exclusivo Windows |

Identificar o servidor web pode revelar versões vulneráveis ou configurações padrão inseguras.

---

## Frameworks Server-Side

| Framework | Linguagem |
|-----------|-----------|
| Laravel | PHP |
| Ruby on Rails | Ruby |
| Spring MVC | Java |
| Django / Flask / FastAPI | Python |
| Express / Sails.js | JavaScript |
| Symfony / Zend | PHP |
