# Protocolo de perguntas

Como conduzir a conversa da ideia até a direção decidida. Vale para a `/prd`. A `/planning` usa a mesma mecânica para as decisões técnicas.

## Mecânica de cada pergunta

- **Use a ferramenta interativa de perguntas** do runtime (no Claude Code, `AskUserQuestion`), que pausa até a resposta. Sem essa ferramenta, faça a pergunta como mensagem completa e **pare**. Nunca responda no lugar do usuário.
- **Uma decisão por pergunta.** Agrupe na mesma chamada (no máximo 4) apenas perguntas realmente independentes. Quando a resposta de uma muda a próxima, pergunte em sequência.
- **Comece com uma recomendação:** diga em uma linha qual opção você escolheria e por quê, para que o usuário reaja a uma posição em vez de um menu em branco.
- **Múltipla escolha sempre que as opções forem previsíveis:** opções rotuladas com a recomendada primeiro, mais uma saída ("Outro — descreva"). Pergunta aberta só quando o espaço de respostas é ilimitado.
- **Feature com muitas dimensões:** uma dimensão por vez ("O que importa primeiro na colaboração? A) Espaços compartilhados B) Presença em tempo real C) Permissões D) Feed de atividade").

## Método do interrogatório

Interrogue até haver entendimento compartilhado: toda decisão de produto que sustenta o PRD resolvida, não um número fixo de perguntas.

- **Mapeie a árvore de decisões:** quais decisões existem e quais dependem de quais.
- **Percorra ramo por ramo:** pergunte primeiro o que destrava mais decisões abaixo.
- **Persiga respostas vagas:** "depende" leva a "depende de quê?"; "provavelmente" precisa ser fechado. Um ramo importante deixado nebuloso volta como retrabalho.
- **Explore antes de perguntar:** se o repositório, a `historia.md` ou a pesquisa já respondem, use a resposta e siga. Perguntas são para incógnitas reais: intenção, prioridade, trade-off.
- **Pare quando a árvore estiver resolvida:** cada ramo decidido ou estacionado em Questões em aberto, com o consentimento do usuário.

## Fases

### 1. Descoberta
- Qual é o problema ou oportunidade central?
- Quem são os usuários afetados?
- O que motivou a iniciativa agora?

### 2. Entendimento
- **O quê:** quais capacidades os usuários precisam?
- **Por quê:** que valor isso gera?
- **Quem:** quais personas e como trabalham hoje?
- O que precisa ser verdade quando isso for entregue, em termos de comportamento, não de métrica?
- Quais restrições existem (conformidade, integrações obrigatórias, privacidade)?
- **Regras de negócio:** invariantes, limites, defaults, permissões e estados. Peça **valores exatos**.

### 3. Refinamento

Depois que a direção foi decidida e o ADR registrado, volte só onde houver ambiguidade real:
- fronteiras de escopo (o que entra e o que o usuário exclui);
- comportamento esperado de cada funcionalidade;
- comportamento nos casos de borda que a varredura levantou e o código não responde.

## Gates de progressão

- Complete ao menos uma rodada de Entendimento antes de decidir a direção.
- Decida a direção só quando todos os ramos dos quais ela depende estiverem resolvidos: propósito, restrições e comportamento esperado.

## Fronteira de foco

As perguntas tratam de **o quê, por quê e quem**. Banco, API, estrutura de código, framework, estratégia de teste, arquitetura e deploy pertencem à `/planning`. Traduza esses assuntos conforme a seção "Foco em negócio" do `SKILL.md`.
