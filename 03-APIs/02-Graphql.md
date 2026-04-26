# GraphQL

## O que é?

GraphQL é uma linguagem de consulta para APIs criada pelo Facebook. Em vez de vários endpoints como no REST, você tem **um único endpoint** e especifica exatamente os dados que quer receber, nem mais, nem menos.

```
REST:  GET /users/1         → retorna tudo do usuário
REST:  GET /users/1/posts   → endpoint separado para os posts

GraphQL: POST /graphql      → um endpoint, você escolhe o que quer
```

---

## Conceitos básicos

### Schema
Define os tipos de dados e a estrutura disponível. É o contrato da API.

### Query (leitura)
Equivale ao `GET` do REST. Busca dados.

```graphql
{
  usuario(id: 1) {
    nome
    email
    posts {
      titulo
    }
  }
}
```

### Mutation (escrita)
Equivale ao `POST`, `PUT`, `DELETE`. Cria ou altera dados.

```graphql
mutation {
  criarUsuario(nome: "João", email: "joao@exemplo.com") {
    id
    nome
  }
}
```

---

## Características importantes para pentest

- **Endpoint único**: geralmente `/graphql`
- **Só aceita `POST`** (na maioria das implementações)
- **Sempre retorna 200 OK**, mesmo quando há erro, o erro está no body da resposta
- Pode expor **toda a estrutura do schema** via Introspection

---

## Localizando o endpoint GraphQL

Tente esses caminhos:
```
/graphql
/graphiql
/graph
/v1/graph
/v1/graphql
/api/graphql
```

---

## Introspection - descobrindo o schema

Uma query de introspection revela toda a estrutura da API: tipos, campos, queries e mutations disponíveis.

```graphql
{
  __schema {
    types {
      name
      fields {
        name
        type {
          name
        }
      }
    }
  }
}
```

**Ferramenta**: **GraphQL Voyager** transforma a resposta de introspection em um diagrama visual interativo.

Se a introspection estiver bloqueada, use **Field Suggestions**, o GraphQL sugere campos similares quando você digita algo errado. A ferramenta **Clairvoyance** automatiza esse processo.

---

## GraphiQL

IDE web para interagir com GraphQL. Se estiver exposta publicamente, é um vetor de reconhecimento enorme.

Caminhos comuns: `/graphiql`, `/graph`, `/api/graphql`

---

## Recursos úteis

- **Burp Suite + InQL Extension** - extensão específica para testar GraphQL
- **GraphQL Network Inspector** - extensão para Chrome DevTools
- **GraphQL Voyager** - visualização do schema
- **Damn Vulnerable GraphQL Application (DVGA)** - lab para praticar
