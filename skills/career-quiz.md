# Skill: Guia de Carreira e Quiz (Career Quiz)

## Visão Geral
Esta skill contém os protocolos metodológicos para conduzir uma avaliação de perfil de carreira (Quiz), definir funções-alvo com base em respostas e orquestrar a delegação de tarefas para subagentes (Scout, Curator, Coach).

## Etapa 1: Fluxo de Inicialização e Quiz
Quando interagir com o usuário pela primeira vez:
1. Verifique se `data/personality-quiz.md` existe e se contém `Concluído: true`.
2. **Se o Quiz for Inexistente**: Crie `data/personality-quiz.md` e inicie a Pergunta 1.
3. **Se o Quiz for Incompleto**: Pergunte se o usuário prefere continuar de onde parou ou recomeçar (se recomeçar, limpe o arquivo).
4. **Se o Quiz estiver Completo**: Vá para a Etapa 3 (Menu Principal).

## Etapa 2: As Perguntas do Quiz (Sequencial)
Faça uma pergunta por vez e aguarde a resposta antes de avançar:
1. Qual área mais te anima? (Opções: Frontend, Backend, Ciência de Dados, Mobile, DevOps, Full Stack, Governança de Dados, Design UX, Design UI, Liderança, RH, Marketing, Growth, Produto, Cibersegurança).
2. Nível de experiência atual? (Júnior, Pleno, Sênior).
3. Preferência de trabalho? (Remoto, Híbrido, Presencial).
4. Localização? (Cidade/Estado ou Remoto).
5. Soft skills mais fortes?
6. Onde você se vê na carreira? (Crescimento técnico, Transição, Primeiro emprego, Liderança).
7. Habilidades técnicas atuais?

*Regra:* O quiz termina apenas quando todas as 7 perguntas forem respondidas. Registre `Concluído: true` no arquivo `data/personality-quiz.md`.

## Etapa 3: Consolidação e Mapeamento
Gere ou atualize o `data/user-profile.md` usando este mapeamento fixo (Área + Nível -> Funções Alvo):
- Frontend + Júnior = Desenvolvedor Frontend, Desenvolvedor UI Júnior, Desenvolvedor Web
- Backend + Pleno = Engenheiro Backend, Desenvolvedor API, Desenvolvedor Python/Java
*(Aplique a lógica equivalente para as outras combinações profissionais, focando em sugerir 3 cargos compatíveis).*

## Etapa 4: Menu Principal e Delegação
Após consolidação, apresente:
1. **A — Buscar vagas**: O Advisor valida se os dados estão completos, empacota o despacho e chama a persona **Scout**.
2. **B — Encontrar cursos**: O Advisor despacha para a persona **Curator**.
3. **C — Praticar entrevista simulada**: O Advisor inicia a delegação sequencial de 6 etapas para a persona **Coach**.
4. **D — Refazer o quiz**: Limpa os dados e recomeça da Etapa 1.

## Tratamento de Handoffs
Ao despachar para os subagentes usando `spawn_agent`:
- Para o **Scout**: Validar `Área de interesse`, `Localização` e `Habilidades atuais` antes de despachar.
- Para o **Curator**: Identificar previamente as lacunas de habilidades do usuário.
- Para o **Coach**: Garantir que as 6 etapas sejam chamadas sequencialmente, validadas a cada retorno de sucesso.
