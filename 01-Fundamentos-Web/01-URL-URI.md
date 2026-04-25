# URL e URI

## O que é uma URL?

URL significa **Uniform Resource Locator** - basicamente, é o endereço de um recurso na internet. Quando você acessa um site, está usando uma URL.

**Exemplo:**
```
https://www.exemplo.com/pagina?chave=valor#secao
```

Quebrando em partes:

| Parte | Exemplo | O que é |
|-------|---------|---------|
| Protocolo | `https://` | Como o recurso vai ser acessado |
| Domínio | `www.exemplo.com` | Onde o recurso está |
| Caminho | `/pagina` | Qual recurso dentro do servidor |
| Parâmetro | `?chave=valor` | Dados extras passados na URL |
| Fragmento | `#secao` | Âncora na página (só no navegador, não vai pro servidor) |

> 💡 O `#` e tudo que vem depois **não é enviado ao servidor**. É algo que o navegador processa localmente.

---

## O que é uma URI?

URI é **Uniform Resource Identifier** - um identificador genérico para qualquer recurso. Toda URL é uma URI, mas nem toda URI é uma URL.

- **URL** - diz onde está e como acessar: `https://exemplo.com/pagina`
- **URN** - só identifica, não diz como acessar: `urn:isbn:0451450523`

---

## Caracteres especiais nas URLs

Alguns caracteres têm funções específicas dentro de uma URL. Entender isso é fundamental para manipulá-las durante um pentest.

| Caractere | Função |
|-----------|--------|
| `?` | Inicia os parâmetros da query string |
| `&` | Separa múltiplos parâmetros: `?a=1&b=2` |
| `=` | Associa chave ao valor: `chave=valor` |
| `%` | Indica URL encoding: `%20` = espaço |
| `#` | Marca um fragmento (âncora) — não vai ao servidor |
| `/` | Separa as partes do caminho |
| `:` | Separa o protocolo (`https:`) ou a porta (`exemplo.com:8080`) |
| `@` | Separa usuário:senha do domínio em URLs com autenticação |
| `+` | Substitui espaço em algumas implementações |
| `;` | Alternativa ao `&` para separar parâmetros |
| `~` | Indica diretório pessoal no servidor |

---

## URL Encoding (Codificação de Porcentagem)

URLs só podem conter caracteres seguros (alfanuméricos e alguns símbolos). Quando há caracteres especiais, eles precisam ser codificados.

A codificação substitui o caractere por `%` seguido do valor hexadecimal do byte.

**Exemplos práticos:**

| Original | Codificado |
|----------|-----------|
| Espaço | `%20` |
| `&` | `%26` |
| `=` | `%3D` |
| `#` | `%23` |
| `/` | `%2F` |
| `@` | `%40` |

**Exemplo real:**
```
Original:  https://exemplo.com/pesquisa?query=termos & condições
Codificado: https://exemplo.com/pesquisa?query=termos%20%26%20condi%C3%A7%C3%B5es
```

### Por que isso importa no pentest?

Durante um teste, é possível usar URL encoding para bypassar filtros que bloqueiam certos caracteres. Se um WAF bloqueia `<script>`, tentar `%3Cscript%3E` pode contornar o filtro.

---

## Construindo URLs para phishing

Com o entendimento de todos esses componentes, dá para criar URLs que parecem legítimas mas redirecionam ou enganam o usuário, usando:
- Subdomínios falsos
- Parâmetros adicionais
- Encoding para disfarçar caracteres

Exemplo de URL maliciosa disfarçada:
```
https://banco-seguro.com.br@attacker.com/login
```
O navegador vai acessar `attacker.com`, mas o usuário vê `banco-seguro.com.br` primeiro.
