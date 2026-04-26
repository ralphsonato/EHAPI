# Entendendo APIs

## O que é uma API?

API significa **Application Programming Interface**, uma interface que permite que aplicações se comuniquem entre si. Diferente de um site web, uma API não retorna HTML para exibição, mas dados estruturados (geralmente JSON ou XML) para serem consumidos por outro sistema.

### Conceitos básicos

- **Endpoint**: uma URL específica para interagir com parte da API
  - `https://api.exemplo.com/v3/users/`
  - `https://api.exemplo.com/v3/products/`
  - `https://api.exemplo.com/orders/status/1/`

- **Recurso singleton**: um objeto específico → `/api/users/{user_id}`
- **Coleção**: grupo de recursos → `/api/users/`
- **Subcoleção**: coleção dentro de um recurso → `/api/users/{id}/settings`

- **Gateway de API**: filtro que valida, monitora e roteia as requisições para o microsserviço correto

---

## Tipos de API

### REST (Representational State Transfer)

O tipo mais comum hoje. Usa os métodos HTTP para operações CRUD:

| Método | Operação |
|--------|---------|
| `GET` | Read (leitura) |
| `POST` | Create (criação) |
| `PUT` | Update completo |
| `PATCH` | Update parcial |
| `DELETE` | Delete |

**6 restrições do design RESTful:**
1. **Interface uniforme** - consistência nas interações
2. **Client/Server separados** - independência entre front e back
3. **Stateless** - cada requisição carrega todas as informações necessárias
4. **Cacheável** - respostas podem ser cacheadas
5. **Sistema em camadas** - pode ter proxies e gateways no meio
6. **Código sob demanda** (opcional) - servidor pode enviar código executável

### SOAP

Mais antigo e verboso. Usa XML e um protocolo próprio (WSDL). Ainda presente em sistemas corporativos legados.

### GraphQL

Linguagem de consulta para APIs. Em vez de múltiplos endpoints, tem **um único endpoint** (`/graphql`) e você especifica exatamente os dados que quer.

---

## Formatos de dados

| Formato | Descrição |
|---------|-----------|
| **JSON** | Pares chave/valor. O mais comum em APIs modernas |
| **XML** | Tags descritivas. Mais verboso, ainda usado em SOAP |
| **YAML** | Legível por humanos, usado em configurações e documentação |

---

## Autenticação em APIs

### Chaves de API (API Keys)
Token único passado no header ou como parâmetro:
```
X-API-Key: abc123xyz
```

### Basic Auth
Usuário:senha em Base64 no header Authorization.

### Bearer Token / JWT
```
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```

### OAuth 2.0
Framework para acesso delegado. Um usuário autoriza um aplicativo a agir em seu nome.

### HMAC (AWS)
Assinatura da requisição usando chave de acesso + chave secreta. Muito usado na AWS.

---

## Headers importantes em APIs

### Headers de autorização
```
Authorization: Bearer <token>
X-API-Key: <chave>
```

### Headers de conteúdo
```
Content-Type: application/json
Accept: application/json
```

### Headers de middleware (X-)
```
X-Response-Time: 123ms        → tempo de processamento
X-Powered-By: Express         → revela o framework (útil para reconhecimento)
X-Rate-Limit-Remaining: 95    → quantas requisições restam
```

---

## Documentação de API

A documentação é um presente para o pentester. Ela revela:
- Todos os endpoints disponíveis
- Parâmetros esperados
- Exemplos de autenticação
- Possíveis ações administrativas

**Padrões de documentação:**
- **OpenAPI/Swagger** → `/swagger-ui/`, `/api-docs/`, `/openapi.json`
- **RAML** → formato YAML para documentação

---
---

# SOA, REST e Microsserviços

## SOA (Service Oriented Architecture)

Arquitetura orientada a serviços. A ideia é dividir o sistema em serviços menores com responsabilidades específicas, que se comunicam via APIs.

Exemplo de serviços granulares:
- `PesquisarProdutos`
- `CriarCompra`
- `CancelarCompra`
- `VerificarEstoque`

---

## Microsserviços

Evolução do SOA. Cada serviço é ainda mais isolado, tem seu próprio banco de dados e pode ser desenvolvido, implantado e escalado de forma independente.

```
[Gateway de API]
      ↓
┌─────────────────────────────────┐
│ Serviço de    Serviço de    Serviço de  │
│ Usuários      Pagamentos    E-mail      │
└─────────────────────────────────┘
```

**Para o pentest:** a comunicação entre microsserviços via API pode ter autenticação mais fraca do que a API pública, pois assume-se que só o sistema interno a acessa. Quando há um SSRF, é possível explorar essas APIs internas.
