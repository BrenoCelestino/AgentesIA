# Skill: Análise e Previsão de Futebol (Copa 2026)

## Visão Geral
Esta skill fornece o protocolo padronizado para buscar estatísticas esportivas, analisar dados e gerar palpites quantitativos precisos para partidas da Copa do Mundo FIFA 2026. Ela também dita as regras para manutenção do estado do campeonato (classificação).

## Ferramentas
1. **Busca Web Integrada (`search_web`) - Ferramenta Principal**
   - Utilizar obrigatoriamente filtros de site para evitar ruído.
   - Sites homologados: `site:ge.globo.com`, `site:uol.com.br`, `site:fifa.com`.
   - Exemplo de Query: `"Brasil vs Noruega Copa do Mundo 2026 escalacao site:ge.globo.com"` ou `"Brasil ultimos jogos resultados site:ge.globo.com"`.
2. **Terminal (Firecrawl CLI) - Fallback/Scraping Profundo**
   - Usar `firecrawl scrape` caso precise do texto completo de uma matéria muito detalhada.

## Protocolo 1: Consulta e Atualização Proativa da Classificação
Antes de realizar qualquer busca na web, o agente DEVE consultar localmente o arquivo `data/classificacao-copa-2026.md` para entender o cenário do torneio:
1. **Consulta Obrigatória**: Leia `data/classificacao-copa-2026.md` antes de qualquer outra ação para obter os dados do chaveamento e a data da última atualização.
2. **Decisão Inteligente de Busca Web**:
   - Compare a data de atualização do arquivo com a data atual do sistema.
   - Se o arquivo já estiver atualizado até a data corrente, ou se o chaveamento/confronto desejado já constar no arquivo como definido, **NÃO** realize buscas na web sobre a tabela ou resultados passados.
   - Caso o arquivo esteja desatualizado (data anterior à atual) ou falte informações cruciais sobre jogos concluídos recentemente, execute a busca: `"resultados copa do mundo ontem site:ge.globo.com"`.
3. **Atualização do Arquivo**: Se encontrar resultados novos na busca, atualize matematicamente a classificação e o chaveamento no arquivo local, modificando também o campo "Data da Última Atualização" para a data atual do sistema.

## Protocolo 2: Análise de Confronto
Quando um confronto for solicitado:

### Etapa 1: Coleta de Dados Quantitativos e Qualitativos
1. **Consultar Primeiro o Arquivo Local**: Verifique a situação das equipes no arquivo `data/classificacao-copa-2026.md`.
2. **Busca Web Focada (Apenas para Informações Faltantes)**:
   - Evite buscar resultados ou posições de tabela que já estejam no arquivo local.
   - Busque apenas dados dinâmicos e qualitativos não contidos no arquivo, tais como: escalações prováveis, desfalques (lesões e suspensões), estatísticas de xG/chutes a gol dos últimos 3 jogos das duas seleções.

### Etapa 2: Aplicação do Mini-Modelo Quantitativo (Inteligência)
1. **Força Ofensiva (Time A)**: Baseado nos gols e criação (xG hipotético) nos últimos 3 jogos.
2. **Vulnerabilidade Defensiva (Time B)**: Gols sofridos e fragilidade recente.
3. Cruza os dados: Se a Força Ofensiva A for significativamente maior que a Vulnerabilidade Defensiva B, o Time A tende a marcar múltiplos gols.
4. Repita para o Time B.

### Etapa 3: Definição dos Palpites
Baseado no modelo, defina:
- **Palpite Principal**: O placar exato mais matematicamente provável (ex: 2x1).
- **Palpite de Segurança**: O resultado genérico de menor risco (ex: Vitória do Time A, ou Dupla Chance A/Empate).

### Etapa 4: Formatação da Saída e Persistência
1. Crie o backup `.bak` do arquivo `data/palpites-copa.md` com timestamp.
2. Salve a análise em `data/palpites-copa.md`.
3. Retorne a análise no seguinte formato numerado (nunca use tabelas markdown):

```md
1. Jogo: [Seleção A] vs [Seleção B]
2. Palpite Principal: [Placar exato]
3. Palpite de Segurança: [Resultado genérico]
4. Justificativa Analítica: [Explique o cruzamento da Força Ofensiva vs Defensiva, xG e impacto de lesões de forma técnica]
5. Confiança: [Alta / Média / Baixa]
```
