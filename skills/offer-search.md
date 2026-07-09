# Skill: Offer Search (Busca de Ofertas com Firecrawl)

## Visão Geral

Capacidade especializada do Caçador de Ofertas para buscar produtos online em múltiplos varejistas via Firecrawl, analisar preço, frete e reputação, e retornar as melhores ofertas ordenadas por valor.

## Ferramentas Utilizadas

1. **Terminal (CLI Firecrawl)**
   - `firecrawl search "[PRODUTO] site:[VAREJISTA]" --json` — busca agregada de produtos
   - `firecrawl scrape <url> --format markdown` — extrai detalhes completos do produto
  - Observação: agentes devem usar o `terminal`/CLI Firecrawl como método primário para buscas ao vivo. Consulte `skills/firecrawlskill.md` para orientações de autenticação, exemplos de comandos e práticas de recuperação de falhas.

2. **Variáveis de Ambiente**
   - `FIRECRAWL_API_KEY` — chave de autenticação Firecrawl (já deve estar definida)

3. **Manipulação de Arquivos**
   - Escrever em `data/busca-ofertas.md` para armazenar histórico e resultados

## Protocolo de Busca

### Etapa 1: Construir Queries de Busca

Input: nome do produto, varejistas alvo
Output: queries Firecrawl estruturadas

**Padrão:** 
```
"[NOME_PRODUTO] site:[DOMINIO_VAREJISTA]"
```

**Exemplos:**
- `"Monitor LG 27 polegadas site:amazon.com.br"`
- `"iPhone 15 site:mercadolivre.com.br"`
- `"Teclado mecânico site:kabum.com.br"`

**Varejistas e seus domínios:**
**Varejistas e seus domínios (validados):**
1. Amazon Brasil: `amazon.com.br`
2. Mercado Livre: `mercadolivre.com.br`
3. Kabum: `kabum.com.br`
4. Americanas: `americanas.com.br`
5. Shopee: `shopee.com.br`
6. Casas Bahia: `casasbahia.com.br`
7. Ponto Frio: `pontofrio.com.br`
8. Extra: `extra.com.br`
9. Fast Shop: `fastshop.com.br`
10. Carrefour: `carrefour.com.br`
11. Pichau: `pichau.com.br`
12. AliExpress: `aliexpress.com`
13. eBay: `ebay.com`

**Observação:** `magazineluiza.com.br` e `walmart.com.br` apresentaram bloqueio ou inconsistência ao tentar extrair detalhes das páginas (scrape) durante a validação e foram descartados da lista ativa. Novos varejistas só devem ser adicionados à lista após validação prévia conforme seção abaixo.

### Etapa 2: Executar `firecrawl search` para Cada Varejista

Comando CLI:
```bash
firecrawl search "[PRODUTO] site:[VAREJISTA]" --json
```

**Resultado esperado:**
```json
{
  "success": true,
  "data": [
    {
      "url": "https://amazon.com.br/s?k=Monitor+LG+27",
      "title": "Monitor LG 27\" UltraGear 144Hz",
      "description": "Tela IPS 27\" 1440p, 144Hz, ideal para gaming...",
      "source": "amazon"
    },
    ...
  ]
}
```

**Tratamento de erro:**
- Se `success: false` ou comando não retornar JSON válido, reportar: `"Falha em firecrawl search para [varejista]: [mensagem de erro]"`
- Se `data` está vazio para um varejista, pular para o próximo varejista (não parar)
- **CONTINUAR** com próximos varejistas mesmo se alguns falharem

### Validação prévia de varejista (OBRIGATÓRIA antes de inclusão)

Antes de incluir um novo varejista na lista ativa, executar estes passos manualmente ou programaticamente:

1. Teste de busca:
  - Comando: `firecrawl search "[PRODUTO_EXEMPLO] site:[DOMINIO_VAREJISTA]" --json`
  - Critério: retorno com `success: true` e pelo menos 1 resultado com URL.
2. Teste de extração (scrape) no top result:
  - Comando: `firecrawl scrape <url_top> --format markdown -o ".firecrawl/test_[varejista].md"`
  - Critério: o conteúdo salvo não deve ser uma página de erro (ex.: 403), não deve conter prompts de login, e deve incluir informações de preço ou conteúdo do produto.
3. Decisão:
  - Se ambos os testes passarem → incluir o varejista na lista ativa.
  - Se o `search` funcionar, mas o `scrape` retornar 403 / conteúdo de login / página vazia → DESCARTAR o varejista para busca via scraping (pode ainda ser consultado via busca genérica/Google).
  - Registrar resultado do teste em `data/agent-activity.log` com: timestamp, varejista, comando executado, status (ok | descartado), motivo.

Exemplo de registro (linha):

```
2026-06-22 10:12:34 | kabum.com.br | search+scrape | ok | detalhes extraídos
2026-06-22 10:13:01 | magazineluiza.com.br | scrape | descartado | 403 no scrape (bloqueado)
```

### Etapa 3: Extrair URLs de Produtos

Das respostas JSON de todos os varejistas, coletar até 2-3 URLs por varejista (total max 15 URLs):

```
urls = [resultado.url para cada resultado em data[:2-3] para cada varejista]
```

### Etapa 4: Executar `firecrawl scrape` em Cada URL

Para cada URL coletada:

```bash
firecrawl scrape <url> --format markdown
```

**Resultado esperado:**
```markdown
# Monitor LG 27" UltraGear 144Hz

## Preço
R$ 1.299,00

## Frete
Frete grátis acima de R$ 150
Estimado: 2-3 dias úteis

## Loja
Amazon Brasil | Rating: 4.8 ⭐ (12.534 reviews)

## Descrição do Produto
[Descrição técnica completa]

## Especificações
- Tamanho: 27"
- Resolução: 2560x1440
- Taxa de Atualização: 144Hz
- ...
```

**Tratamento de erro:**
- Se `firecrawl scrape` falhar para uma URL específica:
  - Usar título e descrição do resultado da busca original
  - Anotar: `"Detalhes completos indisponíveis"`
  - **CONTINUAR** com próxima URL

  ### Fallback: Busca genérica com search_web (quando varejistas ou firecrawl falham)
  
  Se a busca via `firecrawl` falhar, retornar erro de timeout ou a quantidade de ofertas válidas for insuficiente, você **DEVE IMEDIATAMENTE** utilizar a ferramenta nativa `search_web` integrada ao seu sistema.
  
  1. Execute `search_web` com a query "[PRODUTO] preço comprar" ou "[PRODUTO] site:[DOMINIO]" para recuperar links de varejistas de forma nativa e rápida.
  2. A ferramenta `search_web` garante uma alternativa robusta e veloz à pesquisa. Extraia os preços diretamente dos resumos retornados pelo `search_web` ou acesse as URLs coletadas.
  3. Prossiga com o pipeline de extração e scoring utilizando as informações consolidadas.

### Etapa 5: Extrair Dados de Preço e Frete

De cada página raspada, extrair:

**Estrutura esperada:**
```
produto = {
  "titulo": [título extraído],
  "varejista": [nome do varejista],
  "url": [URL original],
  "preco_base": [valor numérico em R$],
  "frete": [valor numérico em R$, 0 se grátis],
  "tempo_entrega_dias": [número de dias],
  "rating": [número 0-5],
  "reviews_count": [número de reviews],
  "disponibilidade": [em estoque | fora de estoque | indisponível],
  "descricao_resumo": [primeiros 200 caracteres]
}
```

**Parsing de valores:**
- Preço: buscar padrão `R$ [número]` ou `R$[número]`
- Frete: buscar `frete grátis` (valor 0) ou `R$ [número]`
- Dias: extrair número de `[N]-[N] dias` ou `[N] dias`
- Rating: buscar estrelas ⭐ ou porcentagem

### Etapa 6: Calcular Score de Oferta

**Algoritmo:**
```
Normalização (0-100):
  preço_normalizado = (preço - min_preço) / (max_preço - min_preço) * 100
  frete_normalizado = (frete - 0) / (max_frete - 0) * 100
  velocidade_score = (tempo_dias - 1) / (tempo_dias_max - 1) * 100
  reputacao_score = (rating / 5) * 100

Score Final = (40 × preço_normalizado) 
            + (30 × frete_normalizado) 
            - (15 × velocidade_score) 
            - (10 × reputacao_score)

Quanto MENOR o score, MELHOR a oferta.
```

### Etapa 7: Filtrar e Ordenar Resultados

**Critérios de exclusão:**
1. Produto com `disponibilidade: "fora de estoque"` → descartar
2. Produto com `disponibilidade: "indisponível"` → descartar
3. Preço absurdamente diferente de outros → pode indicar erro de parsing, validar

**Ordenação:**
1. Ordenar por score (menor primeiro)
2. Se score igual, preferir maior rating
3. Limitar a 3 ofertas vencedoras

### Etapa 8: Compilar Resultado

**Estrutura de dados:**
```
resultado = {
  "timestamp": [YYYY-MM-DD HH:MM:SS],
  "produto_procurado": [termo de busca],
  "ofertas": [
    {
      "rank": 1,
      "varejista": [nome],
      "titulo": [título],
      "preco_base": R$ [valor],
      "frete": R$ [valor],
      "custo_total": R$ [valor_base + frete],
      "tempo_entrega": [N] dias,
      "rating": [X.X] ⭐,
      "reviews": [número],
      "url": [URL],
      "score": [valor numérico]
    },
    ...
  ],
  "melhor_oferta": [rank 1],
  "economia_vs_pior": R$ [diferença]
}
```

## Exemplo de Execução Completa

```
Input: 
  Produto: "Monitor LG 27 polegadas 144Hz"
  Restrições: Máximo R$ 1.500, frete grátis preferível

Etapa 1: Construir queries
  - "Monitor LG 27 144Hz site:amazon.com.br"
  - "Monitor LG 27 144Hz site:mercadolivre.com.br"
  - "Monitor LG 27 144Hz site:kabum.com.br"
  [... mais varejistas ...]

Etapa 2: firecrawl search [query1] ... [query9]
  Resultados: 9 varejistas ✓, 0 falhas

Etapa 3: Coletar URLs
  Total de 18 URLs coletadas (2-3 por varejista)

Etapa 4: firecrawl scrape [url1] ... [url18]
  Sucesso: 16 URLs, Falha: 2 URLs (timeout)

Etapa 5: Extrair dados
  16 produtos extraídos com sucesso

Etapa 6: Calcular scores
  Score mínimo: 32 (Kabum)
  Score máximo: 89 (AliExpress)

Etapa 7: Filtrar e ordenar
  3 ofertas válidas após filtro

Output:
  🥇 Kabum: R$ 1.199 + R$ 45 frete = R$ 1.244 (Score: 32)
  🥈 Amazon: R$ 1.249 + R$ 0 frete = R$ 1.249 (Score: 38)
  🥉 Mercado Livre: R$ 1.179 + R$ 89 frete = R$ 1.268 (Score: 45)
  
  Economia vs pior oferta: R$ 65
```

## Tratamento de Erros

### Cenário 1: Todos os varejistas falhando

```
Estado: erro
Mensagem: Não foi possível pesquisar em nenhum varejista
Erros:
  - Todas as buscas Firecrawl falharam
  - Verifique sua conexão ou tente novamente
```

**Ação:** Reportar ao usuário e retornar ao menu.

### Cenário 2: Alguns varejistas falhando

```
Estado: sucesso
Mensagem: Pesquisaram em 8 de 9 varejistas
Ofertas encontradas: 3

Aviso: 
  - Firecrawl timeout em AliExpress (continuar sem esse resultado)
```

### Cenário 3: Nenhuma oferta encontrada

```
Estado: sucesso
Mensagem: Nenhum produto encontrado
Sugestão: Tente termos menos específicos ou um varejista diferente

Exemplo:
  - Ao invés de: "Monitor LG 27 polegadas 144Hz"
  - Tente: "Monitor 27 polegadas gaming"
```

### Cenário 4: Discrepâncias de Preço

Se o preço extraído parecer anomalia (ex: 10x mais caro que outros):
- Marcar com ⚠️ na exibição
- Anotar: `"Valor pode estar incorreto"`
- Incluir na lista mas alertar o usuário

## Regras Operacionais

1. **Nunca inventar dados** — Se extração falhar, usar dados disponíveis ou anotar indisponibilidade.
2. **Continuar em falhas parciais** — Se um varejista falha, continuar com os próximos.
3. **Case-insensitive** — Todas as comparações devem ser normalizadas.
4. **Limite de ofertas** — Retornar máximo 3 ofertas no resultado final.
5. **Sem scripts Python** — Implementar tudo via Firecrawl CLI.
6. **Timestamps** — Incluir sempre a data/hora da busca.
7. **Sem tabelas Markdown** — Usar listas numeradas com pares chave-valor.

## Integração com Caçador de Ofertas

O Caçador de Ofertas:
1. Recebe solicitação do usuário (produto + restrições)
2. Segue este protocolo de busca
3. Obtém resultado com top 3 ofertas
4. Salva em `data/busca-ofertas.md`
5. Exibe ao usuário em formato legível
6. Oferece opções (A/B/C para abrir links, D para refinar, E para sair)

## Campos Obrigatórios na Resposta

- `timestamp` — data/hora da busca
- `produto_procurado` — termo pesquisado
- `ofertas` — array com top 3 (ou menos se não houver)
- `melhor_oferta` — rank da melhor (1, 2 ou 3)
- `economia_vs_pior` — diferença em R$
- `erros` — array de erros (ou [] se nenhum)
