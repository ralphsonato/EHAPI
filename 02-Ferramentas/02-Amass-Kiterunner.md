# API Mapping - Amass e Kiterunner

## Por que fazer mapeamento de API?

Antes de testar qualquer coisa, você precisa saber o que existe. O reconhecimento é a fase mais importante de um pentest. Quanto mais endpoints e subdomínios você mapear, maior a superfície de ataque para investigar.

---

## OWASP Amass

Ferramenta de reconhecimento passivo e ativo para enumeração de subdomínios e mapeamento de rede.

### Instalação e configuração

```bash
# Baixar o template de configuração
curl https://raw.githubusercontent.com/OWASP/Amass/master/examples/config.ini > $HOME/.config/amass/config.ini
```

Configure chaves de API de fontes como GitHub, Censys e VirusTotal para melhorar os resultados.

### Comandos principais

```bash
# Enumeração passiva (sem contato direto com o alvo)
amass enum -passive -d alvo.com -config config.ini

# Enumeração ativa (com contato direto, tenta transferência de zona, etc.)
amass enum -active -d alvo.com -config config.ini

# Com brute force usando wordlist
amass enum -active -brute -w /usr/share/wordlists/API_superlist -d alvo.com -dir ./resultados

# Pesquisa WHOIS reverso e certificados SSL
amass intel -d alvo.com --whois

# ASN de uma organização
amass intel -org "Nome da Empresa"

# Visualizar relações entre os dados coletados
amass viz -enum -d3 -dir ./resultados
```

### Subcomandos

| Comando | O que faz |
|---------|-----------|
| `amass intel` | Coleta certificados SSL, WHOIS, ASNs |
| `amass enum` | Enumeração de subdomínios |
| `amass viz` | Visualiza os resultados graficamente |
| `amass track` | Compara enumerações ao longo do tempo |
| `amass db` | Gerencia o banco de dados de resultados |

---

## Kiterunner

Ferramenta de descoberta de endpoints de API, mais eficiente que ferramentas genéricas como dirbuster/gobuster porque foi criada especificamente para APIs.

### Vantagens sobre outras ferramentas

- Detecta assinaturas de diferentes frameworks de API (Django, Express, FastAPI, Flask, Spring, etc.)
- Suporta wordlists em formato `.kite`, Swagger JSON e `.txt`
- Muito mais rápido e preciso para APIs do que ferramentas web genéricas

### Instalação

```bash
apt-get install golang
cd /opt
git clone https://github.com/assetnote/kiterunner.git
cd kiterunner
make build
sudo ln -s $(pwd)/dist/kr /usr/local/bin/kr
```

### Baixando wordlists

```bash
# Wordlist de rotas de API do HTTP Archive
curl https://wordlists-cdn.assetnote.io/data/automated/httparchive_apiroutes_2021_06_28.txt > latest_api_wordlist.txt

# Todas as wordlists disponíveis
wget -r --no-parent -R "index.html*" https://wordlists-cdn.assetnote.io/data/ -nH
```

### Uso básico

```bash
# Scan usando wordlist .kite
kr scan https://api.alvo.com -w routes-small.kite

# Scan com wordlist .txt
kr scan https://api.alvo.com -w latest_api_wordlist.txt

# Scan com mais threads (mais rápido)
kr scan https://api.alvo.com -w routes-small.kite -x 20
```

---

## Fluxo de reconhecimento sugerido

1. **Amass** para descobrir subdomínios → `api.alvo.com`, `dev.alvo.com`, `v2.alvo.com`
2. **Kiterunner** em cada subdomínio para descobrir endpoints específicos de API
3. **Postman + Burp** para explorar e testar os endpoints encontrados
