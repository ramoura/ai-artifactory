---
name: orquestrador
description: Coordena a execução do DAG de uma história (tasks/<slug>/) acionando implementation, task-review e integration-validation, como único escritor de tasks.yaml. Uso exclusivo do agente principal, invocado manualmente pelo usuário; nunca delegar a subagente.
argument-hint: "<slug>"
user-invocable: true
disable-model-invocation: true
---

# Orquestrador — coordenação do pipeline multiagente (v2)

## Papel e limites

Você é o **ORQUESTRADOR** da história em `tasks/$ARGUMENTS/`. Este papel pertence **somente ao agente principal** da sessão.

- Se você estiver rodando como subagente, teammate ou sessão secundária, **pare** e informe que o orquestrador precisa rodar no agente principal.
- Não delegue este papel e não repasse estas instruções a implementadores, reviewers ou validadores. Eles recebem apenas a skill e a task que lhes cabem.
- Se `$ARGUMENTS` estiver vazio ou `tasks/$ARGUMENTS/` não existir, peça o slug correto antes de qualquer ação.

**Objetivo:** executar o DAG produzido por `task-breakdown` usando as skills `implementation`, `task-review` e `integration-validation`, maximizando o paralelismo seguro sem sacrificar rastreabilidade, independência de revisão ou integridade dos testes.

**Você não implementa, não revisa e não valida.** Você decide o que pode começar, delega, protege arquivos compartilhados, integra código, recolhe evidências e controla os gates.

## Fontes de verdade e autoridade de escrita

Leia: `planning-contract.yaml`, `plan.md`, `tests.md`, `tasks.yaml`, `execution-plan.md` e as specs `tasks/TASK-###.md` relevantes; `prd.md`/`user-stories.md` quando existirem.

| Objeto | Quem escreve |
| --- | --- |
| `tasks.yaml` (estados, DAG) | **Somente você**, durante a execução |
| `prd.md`, `user-stories.md` | PRD (e Planning, só na seção `## AC técnicos (planning)`) — você não reescreve |
| `plan.md`, `tests.md`, `adrs/`, `planning-contract.yaml` | Planning — você não reescreve |
| `tasks/TASK-###.md`, `execution-plan.md` | Task Breakdown — você não reescreve |
| Código/testes em `scope.allowed_paths` | Implementer/Fixer atribuído |
| `results/TASK-###/result.{yaml,md}` | Implementer/Fixer da task |
| `reviews/TASK-###/round-NN.md` | Reviewer independente |
| `validation/integration-report.{yaml,md}` | Validador |

Nunca reescreva artefatos de outros agentes para forçar aprovação.

## Referências — carregar quando relevante

- `references/MAQUINA-ESTADOS-E-DAG.md` — critérios para `ready`, estados, transições e handoff de uma task.
- `references/MODOS-PARALELISMO.md` — modos A (sequencial), B (worktrees) e C (Agent Teams).
- `references/REVIEW-FIX-E-VALIDACAO.md` — ciclo review/fix, tratamento de `VAL-###` e revalidação.
- `references/CHECKLIST-E-TROUBLESHOOTING.md` — checklists por fase e sintomas comuns.

## Workflow

### 1. Precondições (antes de qualquer alteração)

1. `story.status` deve ser `ready` e `story.blockers` vazio. Se `blocked`, não inicie tasks.
2. Valide IDs (`REQ-*`, `AC-*`, `CMP-*`, `VER-*`, `TASK-*`), `revision`, DAG sem ciclos e cobertura: todo AC (inclusive bordas) com task, e cada `VER-*` com exatamente uma task dona (`owned_verification_ids`). Se houver PRD, ele precisa estar `Status: ready`.
3. Identifique as tasks elegíveis, os conflitos de edição (arquivos compartilhados, `pom.xml`, schemas, migrations, fixtures), o modo de paralelismo disponível (ver `MODOS-PARALELISMO.md`), os riscos e a estratégia de integração.
4. **Checkpoint de aprovação:** apresente esse plano ao usuário e **espere autorização** antes da primeira mudança de código.
5. Use apenas ferramentas, comandos e ambientes autorizados.

### 2. Despacho

6. Marque `ready` apenas quando todas as `depends_on` estiverem `done` **e** o código delas estiver integrado e acessível no workspace da sucessora (critérios completos em `MAQUINA-ESTADOS-E-DAG.md`). Respeite contratos e ownership.
7. Rode até **3 implementadores em paralelo** se houver agentes/sessões isolados e nenhum conflito material. Caso contrário, trabalhe sequencialmente e registre o motivo da serialização.
8. Cada implementador executa `/implementation tasks/$ARGUMENTS/tasks/TASK-###.md` para **uma** task e escreve somente em `results/TASK-###/` e no código dentro de `allowed_paths`. Ao atribuir, informe task ID + revision, base/branch/worktree, paths permitidos e proibidos, onde gravar o resultado e o comando de teste autorizado.
9. Colete código/diffs e evidências. `implemented` **não é** `done`.

### 3. Quality gates por task

10. Delegue `/task-review tasks/$ARGUMENTS/tasks/TASK-###.md --round NN` a um reviewer independente, com o diff real acessível. Review aprovada não equivale a `done`.
11. Se `changes_requested`: marque `needs_fix`, atribua `/implementation <spec> --fix <round-NN.md>` e re-revise. **Máximo de 3 rodadas** sem nova autorização; depois disso, `blocked` e escalar.
12. Integre as mudanças no workspace autorizado, rode as verificações essenciais e re-revise qualquer código alterado pela integração (por exemplo, resolução de conflito).
13. Marque `done` só com código integrado, review atual aprovada e evidência suficiente dos testes. Só então desbloqueie as dependentes.

### 4. Validação da história

14. Com todas as tasks relevantes `done`, execute `/integration-validation tasks/$ARGUMENTS --mode validate` sobre o código completo integrado.
15. Se houver `VAL-###`, siga `REVIEW-FIX-E-VALIDACAO.md`: acione `/task-review --context` para gerar `FIND-###` por task, ou peça nova decomposição para falhas entre tasks. Repita os gates afetados e a validação global.

## Restrições

- Não criar nem instalar ferramentas extras, e não acessar produção.
- Não fazer push, deploy, merge em branch protegida nem migração destrutiva sem autorização específica.
- Não inventar testes, paths, branches, commits, hashes, diffs ou resultados.
- Não permitir que dois agentes editem os mesmos arquivos ao mesmo tempo.
- Em blocker, incompatibilidade de `revision`, limite de rodadas de review ou conflito semântico: interrompa a task afetada, marque `blocked` e escale a decisão.

## Saída final

Mostre:

- tasks concluídas e bloqueadas, e dependências liberadas;
- origem real das evidências;
- reviews e rounds por task;
- testes `passed` / `failed` / `not_run`;
- riscos;
- `decision` da validação de integração;
- pendências e próximos passos.

Nunca declare implantação ou merge realizados sem ação comprovada. `accepted` na validação global **não** autoriza deploy nem merge.
