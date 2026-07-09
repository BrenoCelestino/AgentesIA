# Skill: Geração de Currículo LaTeX (LaTeX CV Generation)

## Visão Geral
Esta skill contém as instruções estruturadas, regras técnicas de LaTeX e metodologias para processar pastas de informações profissionais do usuário, identificar lacunas e gerar currículos LaTeX profissionais otimizados para ATS.

## 1. Processamento da Pasta do Usuário
Ao receber o caminho de uma pasta com dados e certificados do usuário, o agente deve:
- Ler os arquivos de texto, PDFs (se puder extrair conteúdo ou texto), markdown ou logs disponíveis na pasta.
- Organizar e mapear as informações de acordo com as seções do modelo:
  - **Dados Pessoais & Links**: Nome, Cidade/Estado, Telefone, E-mail, LinkedIn, GitHub, Portfólio.
  - **Resumo Profissional**: Anos de experiência, principais tecnologias/conquistas e objetivos.
  - **Experiências**: Cargos, empresas, datas, responsabilidades e, principalmente, resultados quantificáveis.
  - **Formação**: Cursos, instituições de ensino, datas.
  - **Projetos**: Títulos de projetos, tecnologias, links de código e o papel desempenhado.
  - **Certificações e Habilidades Técnicas**: Certificados obtidos (instituição e ano) e categorias de ferramentas.

## 2. Metodologia de Entrevista Iterativa (Resolução de Lacunas)
Se as informações da pasta forem insuficientes para preencher **100%** do template LaTeX ou se puderem ser otimizadas para recrutadores/ATS:
- O agente **DEVE** formular perguntas diretas e assertivas para o usuário.
- O agente não deve fazer todas as perguntas de uma vez se for confuso, mas sim agrupá-las por seções lógica (ex: primeiro confirmar dados pessoais e de contato, depois experiências e resultados, depois projetos e certificados).
- Perguntas essenciais para ATS:
  - *"Você tem métricas quantificáveis de sucesso nessa experiência (ex: reduziu custos, aumentou acessos, acelerou entregas em x%)?"*
  - *"Quais são os links do GitHub/LinkedIn corretos?"*
  - *"Qual é o mês e ano de início e fim da experiência X?"*
  - *"Qual o seu nível de proficiência em inglês/outros idiomas?"*

## 3. Regras Técnicas de Escapes de LaTeX (Crítico)
Para evitar que o arquivo gerado quebre na compilação, o agente deve obrigatoriamente higienizar todos os textos de entrada (substituindo caracteres especiais do LaTeX pelos seus equivalentes escapados):

- `%` $\rightarrow$ `\%` (Comentário)
- `&` $\rightarrow$ `\&` (Alinhamento/Separador de colunas)
- `$` $\rightarrow$ `\$` (Expressão matemática)
- `_` $\rightarrow$ `\_` (Underscore em nomes de variáveis/arquivos)
- `{` $\rightarrow$ `\{`
- `}` $\rightarrow$ `\}`
- `#` $\rightarrow$ `\#`
- `~` $\rightarrow$ `\textasciitilde{}`
- `^` $\rightarrow$ `\textasciicircum{}`
- `\` $\rightarrow$ `\textbackslash{}`

> [!WARNING]
> Caracteres como `_` e `&` são extremamente comuns em links de GitHub/LinkedIn, nomes de variáveis e perfis. Se não forem escapados (ou se não estiverem envoltos por comandos como `\href{url}{texto}` que já tratam URLs adequadamente), a compilação do arquivo `.tex` falhará catastroficamente.

## 4. Otimização ATS (Applicant Tracking Systems)
Ao preencher as experiências, o agente deve reescrever as descrições (mantendo a verdade factual) usando a fórmula:
> **[Verbo de Ação] + [Tarefa/Responsabilidade] + [Resultado Quantificado / Tecnologia Usada]**

*Exemplo não otimizado:* "Fui responsável por fazer o dashboard de vendas."
*Exemplo otimizado para ATS:* "Desenvolveu dashboards interativos utilizando Tableau e SQL, reduzindo o tempo de relatórios semanais da equipe executiva em 30%."

## 5. Estrutura de Arquivo de Saída
O arquivo de currículo final gerado deve ser salvo em:
`data/curriculo/curriculo_<timestamp>.tex` (onde o timestamp é a data do dia em formato YYYYMMDD ou YYYYMMDD_HHMMSS).
O conteúdo do arquivo deve ser apenas código LaTeX válido e compilável.
O agente deve informar ao usuário o caminho exato do arquivo gerado usando links `file://`.

## 6. Resistência a Erros de LaTeX e Acesso Restrito
- **Sintaxe e Comandos LaTeX Estritos:**
  - O agente deve usar apenas comandos e estruturas LaTeX formalmente suportados pelos pacotes declarados em `Modelos/Modelo_LaTeX_Generico_Curriculo_Otimizado.txt` (por exemplo, `\titlerule`, `\resumeItem`, `\resumeSubheading` etc.).
  - É proibido abreviar nomes de comandos ou introduzir novas macros que não estejam devidamente importadas no template.
  - Toda estrutura deve ser fechada corretamente (por exemplo, todo `\begin{itemize}` deve ter seu respectivo `\end{itemize}`).
- **Garantia de Não-Modificação da Pasta do Usuário:**
  - A pasta de dados e certificados do usuário deve ser acessada de forma estritamente passiva (Somente Leitura). 
  - O agente está terminantemente impedido de criar arquivos, fazer edições, renomeações, exclusões ou movimentações em qualquer arquivo ou pasta que pertença ao diretório indicado pelo usuário.

