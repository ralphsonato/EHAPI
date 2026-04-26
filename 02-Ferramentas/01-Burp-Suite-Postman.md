# Burp Suite

## O que é?

O Burp Suite é a ferramenta mais usada em testes de segurança em aplicações web. Funciona como um **proxy** que fica entre o navegador e o servidor, permitindo interceptar, inspecionar e modificar qualquer requisição e resposta HTTP.

---

## Como funciona como proxy?

```
[Navegador] → [Burp Suite :8080] → [Internet/Servidor]
```

Você configura o navegador para usar `localhost:8080` como proxy. A partir daí, todo o tráfego passa pelo Burp antes de chegar ao destino.

### Configurar no navegador
- Proxy: `127.0.0.1`
- Porta: `8080`

### Instalar o certificado CA do Burp

Para interceptar HTTPS, o Burp assina os certificados com sua própria CA. Você precisa instalar essa CA no navegador:

1. Com o Burp rodando, acesse `http://burpsuite`
2. Baixe o certificado `.cer`
3. Instale no navegador como CA confiável

> Alternativamente, use o **Burp Embedded Browser**, ele já vem configurado com o certificado instalado.

---

## Principais ferramentas do Burp Suite

### Proxy
Intercepta requisições em tempo real. O botão **Intercept** pausa cada requisição para você inspecionar e modificar antes de liberar.

### Repeater
Pega uma requisição e permite reenviar quantas vezes quiser, editando parâmetros, headers, body. Essencial para testar vulnerabilidades manualmente.

### Intruder
Automatiza ataques em requisições. Você marca os pontos de injeção e configura um payload. Útil para:
- Brute force
- Enumeração de IDs
- Fuzzing de parâmetros

### Scanner (Pro)
Faz varredura automática de vulnerabilidades nas requisições que passam pelo proxy.

### Spider
Crawleia a aplicação automaticamente descobrindo páginas e endpoints.

### Sequencer
Analisa a aleatoriedade de tokens de sessão para identificar padrões previsíveis.

### Decoder
Codifica e decodifica dados em vários formatos: Base64, URL, HTML, Hex, etc.

### Comparer
Compara duas requisições ou respostas para identificar diferenças relevantes.

---

## Dica: Burp + Postman juntos

Configure o Postman para usar o Burp como proxy:
- Postman Settings → Proxy → `127.0.0.1:8080`

Assim você cria as requisições no Postman e intercepta/modifica no Burp. O melhor dos dois mundos.

---
---

# Postman

## O que é?

O Postman é um cliente de API que permite criar, testar e automatizar requisições HTTP. Pense nele como um navegador especializado em APIs.

Suporta: **REST**, **SOAP** e **GraphQL**.

---

## Request Builder

A interface principal para construir requisições. Você define:
- Método HTTP (GET, POST, PUT, DELETE...)
- URL e parâmetros
- Headers
- Body (JSON, form-data, raw...)
- Autenticação

---

## Pre-request Script

Código JavaScript que roda **antes** de cada requisição. Útil para:
- Configurar variáveis de ambiente
- Gerar valores dinâmicos (timestamp, hash)
- Modificar parâmetros automaticamente

```javascript
// Exemplo: gerar um timestamp antes de enviar
pm.environment.set('timestamp', Date.now());
```

---

## Post-request Script (Tests)

Código JavaScript que roda **depois** de receber a resposta. Útil para:
- Extrair tokens da resposta e salvar como variável
- Encadear requisições (pegar o resultado de uma para usar na próxima)
- Validar respostas automaticamente

```javascript
// Exemplo: salvar token da resposta para usar nas próximas requisições
const resposta = pm.response.json();
pm.environment.set('auth_token', resposta.token);
```

---

## Collection Runner

Executa um conjunto de requisições em sequência automaticamente. Abre possibilidades ofensivas:
- **Brute force** automatizado
- **Enumeração** de endpoints
- Testes encadeados que dependem uns dos outros

---

## Variáveis de Ambiente

Permitem reutilizar valores entre requisições sem copiar e colar:

```
{{base_url}}/api/users
{{auth_token}}
```

Você define `base_url = https://alvo.com` e todas as requisições usam automaticamente.
