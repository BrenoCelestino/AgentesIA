# Skill: Protocolo de Despacho e Handoff

## Objetivo
Definir como o Maestro deve despachar tarefas para subagentes e consolidar respostas, com formato estrito para modelos MoE.

## Regras Globais do Protocolo
1. Todo caminho de arquivo deve ser relativo à raiz do projeto e iniciar com `data/` quando apontar para estado.
2. Nunca usar tabelas markdown em respostas dos agentes. Usar listas numeradas com pares chave-valor.
3. Se qualquer ferramenta falhar, retornar estado `erro`, preencher campo `erros` com a falha exata e interromper o fluxo da tarefa afetada.
4. Nunca inventar dados.
5. Maestro deve sempre incluir o conteúdo completo da persona do agente no envelope de despacho.

## Tabela de Roteamento (formato lista)
1. Chave: A | Agente: SCOUT | Responsabilidade: Buscar vagas
2. Chave: B | Agente: CURATOR | Responsabilidade: Encontrar cursos para lacunas de habilidades
3. Chave: C | Agente: COACH | Responsabilidade: Entrevista simulada
4. Chave: D | Agente: MAESTRO | Responsabilidade: Conduzir ou refazer quiz e regenerar `data/user-profile.md`

## Envelope de Despacho (Maestro -> spawn_agent)
Use exatamente este formato:

```md
## DESPACHO: [NOME_DO_AGENTE]
### referencia_persona
[Conteúdo completo de personas/<nome_do_agente_minusculo>.md]

### tarefa
[Uma frase descrevendo o que o agente deve fazer]

### perfil_usuario
[Conteúdo de data/user-profile.md]

### contexto
[Contexto específico: ex: qual vaga para entrevistar, quais habilidades buscar cursos]

### saida_esperada
[Exatamente em que formato o agente deve retornar]
```

## Envelope de Resposta (Agente -> Maestro)
Use exatamente este formato:

```md
## RESPOSTA: [NOME_DO_AGENTE]
### estado
[sucesso | erro]

### resumo
[Resumo legível de 2-3 frases para o usuário]

### dados
[Resultados como listas numeradas com pares chave-valor. Sem tabelas markdown.]

### erros
[Apenas se estado for erro: o que deu errado]
```

## Especificações de Handoff por Agente
1. SCOUT
1. entrada-obrigatoria: `data/user-profile.md` completo
2. contexto-minimo: fontes alvo, filtros de localização, senioridade
3. saida-minima: lista numerada de vagas com pares chave-valor
4. erro: incluir falha de busca/fonte em `erros`

2. CURATOR
1. entrada-obrigatoria: `data/user-profile.md` completo
2. contexto-minimo: habilidades faltantes priorizadas
3. saida-minima: lista numerada de cursos com pares chave-valor
4. erro: incluir falha de busca/fonte em `erros`

3. COACH
1. entrada-obrigatoria: `data/user-profile.md` completo
2. contexto-minimo: vaga alvo e nível esperado
3. saida-minima: bloco de entrevista da etapa atual com instruções claras
4. erro: incluir falha da etapa em `erros`

4. MAESTRO
1. entrada-obrigatoria: `data/personality-quiz.md`
2. contexto-minimo: status de conclusão e intenção do usuário
3. saida-minima: menu com opções A/B/C/D ou próxima pergunta do quiz
4. erro: incluir falha de leitura/escrita em `erros`

## Despacho Sequencial do COACH (6 despachos)
Quando usuário selecionar C, o Maestro deve executar 6 despachos sequenciais para o COACH:
1. etapa-1: Contexto da entrevista e alinhamento da vaga
2. etapa-2: Perguntas comportamentais
3. etapa-3: Perguntas técnicas
4. etapa-4: Perguntas situacionais
5. etapa-5: Feedback estruturado (pontos fortes e lacunas)
6. etapa-6: Plano de melhoria acionável

Regra de sequência:
1. Só avançar para a próxima etapa se a etapa atual retornar `estado: sucesso`.
2. Se qualquer etapa retornar `estado: erro`, interromper sequência e exibir erro ao usuário.

## Regras de Tratamento de Erros
1. Falha de ferramenta
1. ação: interromper tarefa atual
2. saída: `estado: erro`
3. `erros`: mensagem exata da falha

2. Falha de leitura de arquivo de estado (`data/personality-quiz.md` ou `data/user-profile.md`)
1. ação: reportar erro e solicitar confirmação para recriar arquivo
2. saída: `estado: erro`

3. Resposta malformada de subagente
1. ação: solicitar reexecução com o envelope correto
2. se persistir: `estado: erro` com motivo

4. Dados insuficientes do perfil
1. ação: redirecionar para fluxo do quiz (opção D) antes de novo despacho
2. saída: aviso objetivo ao usuário e próximo passo único
