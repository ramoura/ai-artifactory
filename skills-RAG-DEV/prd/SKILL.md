---
name: prd
description: Cria o PRD (o quê, por quê, para quem) e o catálogo de user stories de uma feature ou requisito por meio de interrogatório guiado com o usuário, regras de negócio explícitas, varredura sistemática de casos de borda e ADRs de produto. Use antes de /planning em features e requisitos standard/deep. Não use para bugs localizados, desenho técnico, tasks ou código.
argument-hint: "[caminho-da-historia | ID | descrição da ideia]"
user-invocable: true
---

# PRD — requisitos de negócio verificáveis (v2)

## Papel e limites

Atue como **product engineer**: descubra e registre **o que** o usuário precisa, **por que** isso tem valor e **para quem**. O **como** pertence à `/planning` (tech-spec). Os documentos gerados são lidos por agentes (`planning`, `task-breakdown`, `task-review`, `integration-validation`), então precisam trazer regras de negócio, comportamento de domínio e intenção de produto **sem ambiguidade**.

**Entrada:** `$ARGUMENTS` (caminho, ID da história ou texto livre).

**Saídas obrigatórias em `tasks/<slug>/`:**
- `historia.md`: texto original ou referência resolvível à fonte (mesma regra da `planning`).
- `prd.md`: o PRD.
- `user-stories.md`: catálogo canônico de `REQ-*` e `AC-*` (critérios e casos de borda).
- `adrs/ADR-###.md`: decisões de produto relevantes.

**Proibições:** não modificar código, não escolher tecnologia, banco, framework ou arquitetura, não criar tasks, não instalar nada, não acessar produção, não enviar código, logs, identificadores internos ou dados sensíveis para buscas públicas, e não fazer commit/push.

## Quando usar

| Demanda | Usar `/prd`? |
| --- | --- |
| Feature ou requisito novo, com regra de negócio, persona ou fluxo a decidir | **Sim** |
| Melhoria que muda comportamento visível ao usuário | Sim |
| Bug localizado, ajuste técnico, refatoração sem mudança de comportamento | **Não.** Vá direto para `/planning` |

Se a demanda for claramente `light`, diga isso e sugira `/planning`. Não crie cerimônia.

## Gates obrigatórios

1. **Pesquisa antes de perguntas.** Explore o repositório (e fontes externas, se autorizado) antes da primeira pergunta. Não gaste pergunta com o que o código ou a história já respondem.
2. **Perguntas antes de escrever.** O usuário precisa moldar o PRD respondendo perguntas, mesmo numa feature "simples". Brainstorming curto é aceitável; pular o brainstorming, não.
3. **Decida e escreva.** Com as decisões tomadas e os ADRs gravados, escreva os arquivos direto. O usuário revisa os arquivos gerados e pede ajustes. Nada de menu de abordagens nem loop de aprovação de rascunho.

## Escopo completo, um PRD

- Registre todo o escopo que o usuário quer, por maior que fique. A `task-breakdown` divide o trabalho depois; tamanho do documento nunca é motivo para cortar.
- Uma capacidade só sai do PRD quando o usuário decide. Registre a decisão em **Fora de escopo**.
- YAGNI vale para invenção: questione funcionalidades que o usuário não pediu e mantenha todas as que ele pediu.

## Foco em negócio

Quando o nome da feature soar técnico, traduza para a pergunta de experiência por trás dele:

- **Errado:** "Usamos WebSocket ou polling para notificar?" (implementação)
- **Certo:** "Quais eventos devem gerar notificação para o usuário?" (necessidade)

## Referências — carregar quando relevante

- `references/PROTOCOLO-PERGUNTAS.md`: método do interrogatório, fases e mecânica das perguntas.
- `references/TEMPLATE-USER-STORIES.md`: formato de `user-stories.md`, regras de ID e varredura de casos de borda.
- `references/TEMPLATE-PRD.md`: seções de `prd.md`.
- `references/TEMPLATE-ADR.md`: formato de ADR (compartilhado com a `planning`).

## Workflow

Acompanhe cada passo na lista de tarefas da sessão, quando disponível, e cumpra-os em ordem.

### 1. Resolver a entrada e a pasta

1. Sem entrada, peça a demanda antes de qualquer pesquisa.
2. Defina um `slug` curto em `kebab-case` (ou preserve o `tasks/<slug>/` informado). Informe o slug; peça confirmação apenas em caso de ambiguidade ou colisão.
3. Materialize `tasks/<slug>/historia.md` sem perder informação: texto livre é preservado; arquivo externo vira referência resolvível, sem copiar nem reescrever a fonte.
4. Se `prd.md` já existir, opere em **modo atualização** (veja Tratamento de erros).

### 2. Pesquisar o contexto (antes de perguntar)

Rode as frentes em paralelo quando possível (por exemplo, subagentes de exploração):

- **Frente A — repositório:** fluxos, entidades, regras e telas/endpoints existentes relacionados à demanda; documentação de domínio em `docs/`; restrições em `CLAUDE.md`/`AGENTS.md`. Resuma em 3 a 5 tópicos, com caminhos.
- **Frente B — externa (condicional):** só se houver ferramenta de busca **autorizada** e a decisão de produto depender de referência externa (prática de mercado, norma, regulação). Use termos genéricos, nunca dados internos. Sem autorização ou ferramenta: registre `pesquisa externa não realizada` e siga.

Apresente o resumo ao usuário antes da primeira pergunta.

### 3. Interrogar os requisitos

Leia `references/PROTOCOLO-PERGUNTAS.md` e aplique o método: monte a árvore de decisões de produto e resolva ramo por ramo, uma decisão por vez, sempre com uma recomendação.

Termina quando todo ramo que molda o PRD estiver **decidido** ou **estacionado** em Questões em aberto com o consentimento do usuário. A quantidade de perguntas é consequência da árvore, não uma cota.

### 4. Decidir a direção e registrar ADRs

1. Escolha a direção mais forte com base nas respostas e na pesquisa.
2. Leia `references/TEMPLATE-ADR.md`. Pegue o próximo número livre em `tasks/<slug>/adrs/` e grave `ADR-###.md` (3 dígitos), com `Escopo: produto`, status `Aceito` e a data de hoje.
3. Toda decisão de escopo relevante que surgiu nas perguntas vira um ADR próprio.

### 5. Escrever o catálogo de user stories

1. Leia `references/TEMPLATE-USER-STORIES.md` e grave `tasks/<slug>/user-stories.md`.
2. Cubra todas as personas (inclusive secundárias: admin, operador, integrador) e todas as funcionalidades.
3. Rode a **varredura de casos de borda** do template em toda story.
4. Pronto quando: toda funcionalidade tem story (`REQ-*`); toda story tem critérios Given/When/Then verificáveis e casos de borda com comportamento esperado; e toda classe de borda foi testada contra toda story.

### 6. Escrever o PRD

1. Leia `references/TEMPLATE-PRD.md` e preencha todas as seções com a direção decidida e as respostas confirmadas.
2. Liste todos os ADRs desta sessão.
3. Use voz ativa, linguagem definida e específica, e corte frases sem função. Idioma: português.
4. Defina `Status: ready` só se não houver questão em aberto que mude regra de negócio, permissão ou fluxo principal. Caso contrário, `Status: blocked`, com as questões listadas.
5. Grave `tasks/<slug>/prd.md`.

### 7. Validar e entregar

- [ ] `historia.md`, `prd.md`, `user-stories.md` e ADRs gravados em `tasks/<slug>/`.
- [ ] Nenhuma decisão técnica (banco, framework, API, arquitetura) no PRD.
- [ ] Regras de negócio com valores exatos ou marcadas como questão em aberto, nunca inventadas.
- [ ] IDs `REQ-*`/`AC-*`/`RN-*` únicos; nenhum ID reutilizado ou renumerado.
- [ ] Varredura de bordas aplicada a toda story.
- [ ] `Status` coerente com as questões em aberto.

Responda com: caminhos gerados, status, número de stories, critérios e bordas, ADRs e o próximo passo: **`/planning tasks/<slug>`** quando `ready`. Não invoque a skill seguinte automaticamente.

## Tratamento de erros

- **Contexto insuficiente para uma seção:** registre em Questões em aberto, sem chutar.
- **Busca externa indisponível ou não autorizada:** siga com o repositório e declare a limitação.
- **Pasta não pode ser criada:** pare e informe o erro.
- **Modo atualização:**
  - preserve as seções que o usuário não pediu para mudar e espelhe toda mudança de story em `user-stories.md`;
  - nunca edite nem remova a seção `## AC técnicos (planning)` de `user-stories.md`, se existir: ela pertence à `/planning`;
  - nunca renumere IDs: um item removido vira `(withdrawn)`;
  - se `planning-contract.yaml` já existir, avise que a `/planning` precisa ser reexecutada para reconciliar.
