# Skill: Busca e Curadoria de Cursos (Course Search)

## Visão Geral
Esta skill orienta a busca em tempo real por capacitação educacional para preencher lacunas técnicas no perfil do usuário, priorizando ferramentas de alta velocidade, como `search_web`, focando em plataformas renomadas de tecnologia.

## Ferramentas (Inteligência e Fallback)
1. **Busca na Web Integrada (`search_web`) - Opção Primária/Rápida**
   - Utilize ativamente filtros focados em plataformas validadas para evitar poluição visual e lentidão. 
   - Exemplos de Plataformas Validadas: `site:udemy.com`, `site:coursera.org`, `site:alura.com.br`, `site:edx.org`.
   - Exemplo de Query Direcionada: `"curso postgresql avancado site:alura.com.br"`.
   - **MUITO IMPORTANTE:** O `search_web` é sua via expressa. Caso o Firecrawl retorne timeout ou bloqueios, use o `search_web` sem hesitar.
2. **Terminal (Firecrawl CLI) - Exploração Completa**
   - Se for requisitado analisar a ementa inteira do curso para garantir compatibilidade milimétrica, rode o `firecrawl scrape`.

## Protocolo de Curadoria Educacional

### Etapa 1: Análise de Lacunas
1. Ler as habilidades técnicas presentes no perfil do candidato (`data/user-profile.md`).
2. Ler as habilidades exigidas pela vaga-alvo (`data/job-search-results.md`), se existir.
3. Subtrair as atuais das requeridas para obter as **lacunas críticas** (habilidades faltantes).

### Etapa 2: Execução da Pesquisa Direcionada
- Para cada lacuna, dispare queries no `search_web` combinadas com o nível do candidato (ex: iniciante, arquitetura avançada) focadas nas plataformas validadas.
- Reúna Título, Plataforma, Resumo e Link de 2 a 5 cursos.

### Etapa 3: Formatação (Response Envelope)
O resultado deve ser formatado em lista numerada no nó `dados` e enviado ao despachante principal:
```md
1. titulo: [Título Real Obtido]
   plataforma: [Plataforma ex: Alura]
   habilidade_alvo: [Lacuna preenchida]
   carga_horaria: [Extraída ou "Não especificada"]
   link: [Link Limpo e Real]
   resumo: [Resumo objetivo sobre como o curso resolve a lacuna de nível X]
```

## Regras de Desempenho
- Nunca invente links, cursos ou ementas. As informações devem ser baseadas puramente na busca em tempo real.
- Sempre tente parear a dificuldade do curso com o nível atual do usuário (Não indique "Introdução ao Python" para um Engenheiro Sênior; busque "Arquitetura Avançada em Python").
