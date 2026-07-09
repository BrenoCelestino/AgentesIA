# Skill: Simulação de Entrevista e Mentoria (Interview Simulation)

## Visão Geral
Esta skill contém a metodologia estruturada para simular entrevistas de emprego, formular perguntas pertinentes (técnicas e comportamentais) baseadas no perfil do candidato, e gerar planos de desenvolvimento e feedback.

## Ferramentas (Inteligência Rápida)
**Busca na Web Integrada (`search_web`)**:
- Se precisar de exemplos reais de perguntas técnicas atualizadas, **utilize imediatamente** o `search_web`. 
- Exemplo: `"perguntas avançadas entrevista Node.js site:github.com"` ou `"top react interview questions 2026"`.
- Não perca tempo com scrapings lentos do Firecrawl para essa finalidade.

## Metodologia de 6 Etapas Sequenciais
O agente deve processar rigorosamente as etapas abaixo. Cada etapa é executada individualmente a cada invocação/despacho e deve registrar o status em `data/coach-feedback.md`.

### Etapa 1: Contexto e Alinhamento
- Apresentar as regras da simulação e ler o `data/user-profile.md` para entender o nível (Júnior/Pleno/Sênior).
- Definir a vaga-alvo para nortear o tom da entrevista.

### Etapa 2: Entrevista Comportamental
- Realizar perguntas baseadas no modelo STAR (Situação, Tarefa, Ação, Resultado).
- Avaliar soft skills do perfil (comunicação, resolução de conflitos).

### Etapa 3: Entrevista Técnica
- Usar a ferramenta de busca (`search_web`) se necessário para buscar perguntas de alto nível condizentes com as *hard skills* lidas.
- A pergunta deve testar profundamente arquitetura, código ou ferramentas, de acordo com o nível exigido.

### Etapa 4: Entrevista Situacional
- Propor um cenário hipotético de pressão (ex: sistema caiu na Black Friday, conflito de PR no Git).
- O objetivo é avaliar a tomada de decisão sob pressão.

### Etapa 5: Feedback Estruturado
- Após receber a resposta da etapa 4, consolidar todas as respostas dadas pelo candidato.
- Formatar o feedback (Pontos Fortes e Lacunas Observadas).

### Etapa 6: Plano de Melhoria Acionável
- Gerar um cronograma prático e tangível (ex: indicações de leitura, 3 metas para a próxima semana).

## Estrutura da Resposta
A cada etapa (Response Envelope), a saída deve incluir no nó "dados":
1. `etapa_concluida`: [Número e nome]
2. `pergunta_avaliada` / `avaliacao_resposta`: Análise crítica e realista do que foi respondido (exija completude, aponte omissões).
3. `proxima_etapa`: [Número e nome da seguinte]
4. `instrucao_proximo_passo`: O que o candidato deve fazer agora.
