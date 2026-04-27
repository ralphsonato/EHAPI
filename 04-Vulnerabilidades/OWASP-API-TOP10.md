# OWASP API Top 10

## Comparativo 2019 vs 2023

| 2019 | 2023 |
|------|------|
| API1: Broken Object Level Authorization | API1: Broken Object Level Authorization |
| API2: Broken Authentication | API2: Broken Authentication |
| API3: Excessive Data Exposure | API3: Broken Object Property Level Authorization |
| API4: Lack of Resources & Rate Limiting | API4: Unrestricted Resource Consumption |
| API5: Broken Function Level Authorization | API5: Broken Function Level Authorization |
| API6: Mass Assignment | API6: Unrestricted Access to Sensitive Business Flows |
| API7: Security Misconfiguration | API7: Server Side Request Forgery |
| API8: Injection | API8: Security Misconfiguration |
| API9: Improper Assets Management | API9: Improper Inventory Management |
| API10: Insufficient Logging & Monitoring | API10: Unsafe Consumption of APIs |

---

## API1 - Broken Object Level Authorization (BOLA / IDOR)

**O que é:** Um usuário consegue acessar dados de outro usuário trocando um identificador na requisição.

**Exemplo:**
```
GET /api/users/123/profile    ← seu perfil
GET /api/users/124/profile    ← perfil de outra pessoa (deveria ser bloqueado)
```

Se trocar o `123` por qualquer outro número funcionar, a API não está verificando quem pode acessar o quê.

**Como testar:** Faça login com dois usuários diferentes. Use os tokens de um para acessar recursos do outro.

---

## API2 - Broken Authentication

**O que é:** Falhas na implementação da autenticação que permitem comprometer contas ou tokens.

**Exemplos:**
- Endpoint de login sem rate limiting → brute force possível
- Tokens JWT com algoritmo `none`
- Tokens que nunca expiram
- Senhas sem complexidade mínima
- Credenciais em URLs

---

## API3 (2023) - Broken Object Property Level Authorization

**O que é:** Exposição ou modificação indevida de propriedades de um objeto.

Engloba o que antes eram dois problemas separados: Excessive Data Exposure e Mass Assignment.

---

## API4 - Unrestricted Resource Consumption (antes: Lack of Rate Limiting)

**O que é:** A API não limita o consumo de recursos por usuário/IP. Permite:
- Brute force de senhas
- Enumeração de usuários
- DoS por esgotamento de recursos
- Abuso de operações custosas (envio de email, geração de relatório)

**Como testar:** Tente fazer centenas de requisições ao mesmo endpoint e observe se há alguma limitação.

---

## API5 - Broken Function Level Authorization

**O que é:** Usuários comuns conseguem acessar funcionalidades administrativas.

**Exemplos:**
- `DELETE /api/users/{id}` acessível por usuário comum
- Endpoint de admin sem verificação de role
- Trocar método HTTP (de `GET` para `DELETE`) sem restrição

---

## API6 (2023) - Unrestricted Access to Sensitive Business Flows

**O que é:** Um atacante pode abusar de funcionalidades legítimas de formas que causam dano ao negócio.

**Exemplos:**
- Criar milhares de contas para abusar de cupons de desconto
- Reservar todos os ingressos de um show sem intenção de comprar
- Automatizar o processo de "curtir" para manipular rankings

---

## API7 (2023) / API8 (2019) - Security Misconfiguration

**O que é:** Configurações padrão inseguras, permissões excessivas, serviços desnecessários expostos.

**Exemplos comuns:**
- Debug mode ativo em produção
- Swagger UI exposto publicamente com endpoints sensíveis
- `/actuator/` exposto (Spring Boot health endpoints)
- CORS configurado como `*`
- Mensagens de erro detalhadas revelando stack traces

**Endpoints que revelam informações:**
```
/swagger-ui/index.html
/actuator/
/api-docs/
/.env
/config.json
```

---

## API8 (2019) - Injection / SQL & NoSQL Injection

**O que é:** Dados não sanitizados são usados diretamente em queries, comandos ou templates.

**Tipos:**
- SQL Injection
- NoSQL Injection (MongoDB, etc.)
- Command Injection
- Path Traversal

**Payload de teste básico:**
```
username: admin' OR '1'='1
username: {"$gt": ""}    ← NoSQL
```

---

## API9 - Improper Inventory / Assets Management

**O que é:** A organização não sabe exatamente quais versões da API estão em produção.

**Exemplos:**
- API v1 ainda ativa com vulnerabilidades já corrigidas na v2
- Ambiente de desenvolvimento acessível publicamente
- Endpoints legados esquecidos sem autenticação

**Como testar:**
```
/api/v1/users     ← tente versões antigas
/api/v2/users
/api/beta/users
/api/internal/users
```

---

## API10 (2019) - Insufficient Logging & Monitoring

**O que é:** A aplicação não registra eventos de segurança relevantes ou não monitora por comportamentos anômalos.

**Impacto:** O atacante pode operar por meses sem ser detectado.

**O que deveria ser logado:**
- Tentativas de autenticação falhas
- Acessos a recursos não autorizados
- Payloads suspeitos
- Mudanças em dados críticos

---

## API10 (2023) - Unsafe Consumption of APIs

**O que é:** A aplicação confia cegamente em dados recebidos de APIs de terceiros, sem validar ou sanitizar.

**Risco:** Se a API de terceiro for comprometida ou retornar dados maliciosos, a aplicação os processa sem questionar.

---
---

# OWASP Top 10 Web - Os principais

## Falha na Autorização (IDOR / BOLA)

Quando um usuário acessa ou modifica dados de outros usuários sem permissão. Geralmente causado por controles de acesso fracos.

**Sintoma:** Trocar um ID na URL ou no body da requisição revela dados de outro usuário.

## Falha na Autorização de Função (BFLA)

Quando um usuário executa ações que deveriam ser restritas a papéis com mais privilégio.

**Sintoma:** Um usuário comum consegue deletar outros usuários ou acessar o painel de admin.

## Exposição Excessiva de Dados

A API retorna mais dados do que o necessário para aquela operação.

**Exemplo:** Uma busca de usuário retorna também a senha hasheada, token interno, dados de pagamento...

**Solução:** A API deve filtrar o output baseado no que o consumidor realmente precisa.

## Injeção de Dados

Entrada do usuário usada diretamente em queries ou comandos sem sanitização.

| Tipo | Alvo | Exemplo de payload |
|------|------|-------------------|
| SQL | Banco relacional | `' OR 1=1--` |
| NoSQL | MongoDB, etc. | `{"$gt": ""}` |
| Path Traversal | Sistema de arquivos | `../../etc/passwd` |
| Command | SO | `; ls -la` |

## Falhas de Configuração de Segurança

- Contas e senhas padrão
- Mensagens de erro detalhadas
- Debug ativo em produção
- Componentes/bibliotecas desatualizados

**Spring4Shell (CVE-2022-22965):** Exemplo de RCE em aplicações Spring Framework rodando no JDK 9+ com Tomcat.

## Mass Assignment

O usuário envia campos extras no body da requisição que não deveriam ser modificáveis.

**Exemplo:**
```json
POST /api/users/profile
{
  "nome": "João",
  "role": "admin"    ← campo que o usuário não deveria poder modificar
}
```

Se a API não filtrar os campos aceitos, o usuário consegue promover a si mesmo para admin.

## Gestão Imprópria de Ativos

Falta de inventário atualizado dos componentes e versões usadas. Componentes desatualizados com vulnerabilidades conhecidas se tornam pontos de entrada.

**Troca de versão de API:** Testar versões antigas (`/v1/`) que podem ter menos controles de segurança do que a versão atual.

## GraphQL - Vulnerabilidades específicas

- **Falhas de autorização** → IDOR via queries
- **Consumo excessivo de recursos** → queries aninhadas em profundidade para DoS
- **SQL/NoSQL Injection** → via argumentos das queries
- **SSRF** → via mutation que faz requisição externa
- **Information Disclosure** → via introspection ou field suggestions
