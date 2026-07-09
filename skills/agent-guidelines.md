# Guia Geral para Agentes — Regras de Uso e Manipulação de Dados

Objetivo: fornecer regras claras e obrigatórias que todos os agentes (Maestro e subagentes) devem seguir ao operar neste repositório, garantindo integridade dos arquivos, segurança de dados e previsibilidade das respostas.

1. Âmbito e Aplicabilidade
- Este guia aplica-se a todos os agentes que operam neste projeto e deve ser carregado/consultado antes de qualquer operação automatizada.
- Arquivos de configuração, personas e skills (pastas `personas/` e `skills/`) são *somente leitura* para agentes: NUNCA altere ou sobrescreva esses arquivos sem autorização explícita do usuário humano.

2. Convenções de Persistência de Estado
- Todo estado persistido por agentes deve estar restrito à pasta `data/`.
- Nomes de arquivos de estado devem ser claros e usar hífen ou underscore, ex.: `data/busca-ofertas.md`, `data/user-profile.md`.
- Ao gravar, respeitar os esquemas e templates já existentes em `data/`. Se um template não existir, reportar erro e pedir autorização para criar o arquivo.
- Ao atualizar um arquivo existente, NÃO sobrescrever o conteúdo sem: (a) criar um backup prévio com sufixo `.bak` e (b) incluir um cabeçalho com timestamp e nome do agente.

3. Permissões e Limitações de Escrita
- Agentes só podem criar/editar arquivos dentro de `data/` e nunca devem criar arquivos em outras pastas do repositório (ex.: `personas/`, `skills/`, `.git/`, raiz do projeto) sem permissão explícita.
- Logs de atividade podem ser adicionados em `data/agent-activity.log` com uma entrada por ação (timestamp, agente, ação, arquivo afetado, resumo). Se o arquivo não existir, pedir confirmação antes de criá-lo.

4. Uso de Ferramentas Externas e CLI
- O uso do Terminal/CLI é permitido quando justificado (ex.: Firecrawl). Priorizar sempre as rotas documentadas em `skills/firecrawlskill.md`.
- Nunca gravar chaves, tokens ou credenciais em arquivos do repositório. Use somente variáveis de ambiente ou cofres externos. Se um segredo for recebido por engano, reportar imediatamente e remover qualquer rastreamento local (mas não escreva o segredo em disco).

6. Validação de Dados e Fonte Única da Verdade
- Não inventar dados. Todas as informações factuais devem derivar de fontes verificáveis (ex.: saídas do Firecrawl, arquivos em `data/`, ou APIs autorizadas).
- Ao consumir outputs de scrapes (ex.: `.firecrawl/*.md`), validar consistência (preço numérico, formato de data) antes de incluir nos `dados` de saída.

7. Tratamento de Erros
- Em qualquer falha de ferramenta, leitura ou escrita, retornar `estado: erro`, preencher `erros` com a mensagem exata da falha e interromper o fluxo da tarefa afetada.
- Se a falha for causada por bloqueio do site (ex.: 403), documentar o código de status no `erros` e tentar fontes alternativas listadas na persona.

8. Interações com Outros Agentes
- Nunca modificar arquivos de persona ou skill de outro agente. Se detectar inconsistência em `personas/` ou `skills/`, reportar ao Maestro com `estado: erro` e descrição do problema.
- Ao despachar subagentes (spawn), usar o Envelope de Despacho definido em `skills/dispatch.md` e sempre incluir o conteúdo completo da persona no campo `referencia_persona`.

9. Segurança e Privacidade
- Não persistir informações sensíveis do usuário (senhas, tokens, documentos pessoais) no repositório. Se necessário armazenar PII, use mecanismos aprovados externamente (não implementados aqui) e peça autorização ao usuário.

10. Auditoria e Transparência
- Toda ação que altere `data/` deve gerar uma entrada de auditoria em `data/agent-activity.log` contendo: timestamp, agente, ação (create/update/append), arquivo alvo, breve justificativa.

11. Boas Práticas de Escrita em Arquivos
- Use Markdown para arquivos em `data/` e respeite os templates existentes.
- Para adicionar resultados, preferir anexar se o histórico for relevante; para substituir, justificar no cabeçalho da alteração.

12. Exceções e Autorizações
- Qualquer exceção às regras acima exige aprovação explícita do usuário humano. Quando uma operação exigir criação/alteração fora das regras, o agente deve retornar `estado: erro` com instruções claras e aguardar autorização.

13. Atualização deste Guia
- Mudanças neste arquivo devem ser feitas manualmente por um humano e registradas no changelog. Agentes NÃO devem modificar este arquivo.