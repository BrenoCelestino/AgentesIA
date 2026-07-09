# Skill: Job Search (Busca de Vagas)

## Visão Geral

Capacidade especializada do Scout para buscar vagas de emprego via Firecrawl, analisar correspondência de habilidades e retornar resultados estruturados.

## Ferramentas Utilizadas

1. **Terminal (CLI Firecrawl)**
   - `firecrawl search "query" --json` — busca vagas agregadas
   - `firecrawl scrape <url> --format markdown` — extrai detalhes completos da vaga
  - Observação: usar o `terminal`/CLI como primeira opção para buscas em tempo real. Consulte `skills/firecrawlskill.md` para práticas recomendadas, autenticação e recuperação de falhas.

2. **Manipulação de Arquivos**
   - Ler `data/user-profile.md` para obter habilidades, nível e localização do usuário
   - Escrever em `data/job-search-results.md` para armazenar resultados

## Protocolo de Busca

### Etapa 1: Construir Query de Busca

Input: área de interesse, localização
Output: query firecrawl estruturada

```
query = "vagas [AREA] [LOCALIZACAO]"
```

**Exemplos:**
- "vagas desenvolvedor frontend Brasília"
- "vagas engenheiro de dados remoto"
- "vagas UX designer São Paulo"

### Etapa 2: Executar `firecrawl search`

Comando CLI:
```bash
firecrawl search "vagas [AREA] [LOCALIZACAO]" --json
```

**Resultado esperado:**
```json
{
  "success": true,
  "data": [
    {
      "url": "https://exemplo.com/vaga/123",
      "title": "Desenvolvedor Frontend Junior",
      "description": "Procuramos um frontend developer...",
      "companyName": "Empresa XYZ",
      "location": "São Paulo, SP",
      "source": "indeed"
    },
    ...
  ]
}
```

**Tratamento de erro:**
- Se `success: false` ou comando não retornar JSON válido, reportar erro exato: `"Falha em firecrawl search: [mensagem de erro]"`
- Se `data` está vazio, reportar: `"Nenhuma vaga encontrada para a busca: [query]"`
- **PARAR e reportar ao Maestro** — não continuar silenciosamente

### Etapa 3: Extrair URLs de Vagas

Das respostas JSON, coletar até 5 URLs para processamento seguinte:

```
urls = [resultado.url para cada resultado em data[:5]]
```

### Etapa 4: Executar `firecrawl scrape` em Cada URL

Para cada URL da etapa 3:

```bash
firecrawl scrape <url> --format markdown
```

**Resultado esperado:**
```markdown
# [Título da Vaga]

## Empresa
[Nome da empresa]

## Localização
[Cidade ou Remoto]

## Descrição
[Descrição completa da vaga]

## Requisitos
[Habilidades e requisitos exigidos]
```

**Tratamento de erro:**
- Se `firecrawl scrape` falhar para uma URL específica:
  - Usar título e descrição do resultado da busca (da etapa 2)
  - Anotar: `"Extração completa indisponível"`
  - **CONTINUAR** com próxima URL (não parar)

### Etapa 5: Extrair Habilidades Requeridas

Da descrição/requisitos da vaga, identificar habilidades técnicas e não-técnicas:

**Palavras-chave a procurar (case-insensitive):**
- Linguagens: Python, JavaScript, Java, C++, C#, PHP, Ruby, Go, Rust, Kotlin, Swift, TypeScript
- Frameworks: React, Vue, Angular, Django, Flask, Spring, Laravel, Express, FastAPI
- Bancos de Dados: SQL, PostgreSQL, MySQL, MongoDB, Redis, Elasticsearch, Firebase
- DevOps/Cloud: AWS, GCP, Azure, Docker, Kubernetes, CI/CD, Jenkins, GitLab
- Outras: Git, API REST, GraphQL, Microserviços, Machine Learning, TensorFlow, Pandas, Figma, XD

**Algoritmo:**
```
habilidades_requeridas = []
para cada palavra-chave em [lista acima]:
  se palavra-chave aparece em (titulo + descricao + requisitos):
    adicionar palavra-chave em habilidades_requeridas
```

**Escrever em lowercase para comparação normalizada.**

### Etapa 6: Corresponder Habilidades do Usuário

Input: `habilidades_atuais` do `data/user-profile.md` (já em lowercase)

**Algoritmo de correspondência:**
```
habilidades_usuario = [normalizar para lowercase cada habilidade em user-profile.md]
habilidades_correspondentes = []
habilidades_faltantes = []

para cada habilidade em habilidades_requeridas:
  se habilidade em habilidades_usuario:
    adicionar em habilidades_correspondentes
  senão:
    adicionar em habilidades_faltantes

contagem = len(habilidades_correspondentes) / len(habilidades_requeridas)
percentual = contagem * 100
```

**Exemplo:**
- Habilidades do usuário: `Python, JavaScript, Git, React`
- Habilidades requeridas: `Python, JavaScript, React, Django, PostgreSQL`
- Correspondentes: `Python, JavaScript, React` (3)
- Faltantes: `Django, PostgreSQL` (2)
- Contagem: `3 de 5 (60%)`

### Etapa 7: Compilar Resultado de Uma Vaga

**Estrutura de dados:**

```
vaga = {
  "titulo": [titulo extraído],
  "empresa": [nome da empresa],
  "localizacao": [localização extraída ou "Remoto"],
  "link": [URL original],
  "habilidades_correspondentes": [lista],
  "habilidades_faltantes": [lista],
  "contagem_correspondencia": "[N de M habilidades]",
  "descricao_resumo": [primeiras 200 caracteres da descrição],
  "nivel_vaga": [Júnior | Pleno | Sênior | Não especificado]
}
```

### Etapa 8: Filtrar e Ordenar Resultados

**Ordem de prioridade:**
1. Vagas com melhor correspondência de habilidades (maior percentual primeiro)
2. Se correspondência igual, vagas do mesmo nível que o usuário
3. Se nível não especificado na vaga, incluir mas marcar como "nível não especificado"

**Limitar a 5 vagas no resultado final.**

### Etapa 9: Retornar Response Envelope

**Formato de resposta:**

```
Estado: sucesso | erro
Mensagem: [breve descrição do resultado]
Erros: [lista de erros, se houver]

Dados:
1. titulo: [título da vaga]
   empresa: [empresa]
   localizacao: [localização]
   link: [URL]
   habilidades_correspondentes: [habilidade1, habilidade2, ...]
   habilidades_faltantes: [habilidade3, habilidade4, ...]
   contagem_correspondencia: [N de M habilidades (P%)]
   resumo: [primeiros 200 caracteres da descrição]
   nivel_vaga: [Júnior | Pleno | Sênior | Não especificado]

2. [próxima vaga no mesmo formato]
```

## Tratamento de Erros

### Cenário 1: Falha em `firecrawl search`

**Ação:** Se o `firecrawl search` falhar (retornar erro ou timeout), você **DEVE IMEDIATAMENTE** utilizar a ferramenta nativa `search_web` integrada ao seu sistema como alternativa (fallback). Use o `search_web` com a mesma query de busca para coletar URLs e prosseguir com o fluxo de extração. Se ambas as opções falharem, reporte o erro ao usuário.
```
Estado: erro
Mensagem: Busca de vagas falhou via Firecrawl e Web Search.
Erros:
  - Falha em firecrawl e search_web: [mensagem exata do erro]
```
Reportar ao Maestro e retornar ao menu apenas se ambas falharem.

### Cenário 2: Nenhuma vaga encontrada

```
Estado: sucesso
Mensagem: Nenhuma vaga encontrada para a busca
Dados: []
Sugestao: Tente ampliar os termos (ex: área mais geral, localização alternativa)
```

### Cenário 3: Falha parcial em `firecrawl scrape`

- Usar dados do resultado da busca original
- Anotar no campo `resumo`: `"[Detalhes indisponíveis. Título: ...]"`
- Continuar com próxima vaga

### Cenário 4: Habilidades do usuário não fornecidas

```
Estado: erro
Mensagem: Perfil de usuário incompleto
Erros:
  - Campo 'Habilidades atuais' vazio em data/user-profile.md
```

## Regras Operacionais

1. **Nunca inventar dados** — Se extração falhar, usar dados disponíveis ou anotar indisponibilidade.
2. **Case-insensitive** — Todas as comparações de habilidades devem ser em lowercase.
3. **Limite de 5 vagas** — Retornar no máximo 5 resultados por busca.
4. **Sem scripts Python** — Implementar tudo via conversação e CLI Firecrawl.
5. **Erros explícitos** — Se algo falhar, incluir mensagem de erro exato e não continuar silenciosamente.
6. **Timestamps** — Incluir data/hora da busca no resultado para rastreabilidade.

## Exemplo de Execução Completa

```
Input: 
  Area: Frontend
  Localizacao: Brasília
  Habilidades do usuário: Python, JavaScript, React, CSS, HTML

Comando 1: firecrawl search "vagas desenvolvedor frontend Brasília" --json
Resultado: 5 vagas encontradas

Comando 2-6: firecrawl scrape [url1] --format markdown ... [url5]
Resultados: 4 extrações com sucesso, 1 falha

Etapa de correspondência:
  Vaga 1: React, JavaScript, CSS, HTML encontrados → 4 de 5 habilidades (80%)
  Vaga 2: React, TypeScript, Node.js → 1 de 3 habilidades (33%)
  ... (mais vagas)

Saída: Top 5 vagas ordenadas por correspondência

Response Envelope:
Estado: sucesso
Mensagem: Encontradas 5 vagas de desenvolvedor frontend em Brasília
Timestamp: 2026-06-22 14:35:00
Dados:
  1. titulo: Desenvolvedor Frontend Senior
     empresa: Tech Corp
     localizacao: Brasília, DF
     link: https://...
     habilidades_correspondentes: React, JavaScript, CSS, HTML
     habilidades_faltantes: TypeScript, Vue
     contagem_correspondencia: 4 de 6 habilidades (67%)
     resumo: Procuramos um developer frontend experiente para...
     nivel_vaga: Sênior

  [... mais vagas ...]
```

## Campos Obrigatórios no Response

- `estado` — sucesso | erro
- `mensagem` — descrição legível
- `timestamp` — data/hora da busca
- `dados` — array de vagas ou []
- `erros` — array de mensagens de erro ou []

## Integração com Maestro

O Maestro:
1. Constrói envelope de despacho com user-profile.md
2. Chama `spawn_agent` com esta skill como referência
3. Recebe Response Envelope do Scout
4. Salva dados em `data/job-search-results.md`
5. Exibe vagas ao usuário em formato legível
6. Retorna ao menu
