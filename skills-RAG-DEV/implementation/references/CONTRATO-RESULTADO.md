# Contrato de resultado de uma task — schema_version 1

A Implementation escreve **somente os seus próprios**:

```
tasks/<slug>/results/TASK-###/
├── result.yaml
└── result.md
```

Se o workspace for worktree isolada, o orquestrador deve **coletar/integrar** os arquivos antes de atualizar o registro canônico. Evite armazenar segredos, dados pessoais e logs integrais nas evidências.

## Exemplo ilustrativo `result.yaml`

```yaml
schema_version: 1
task_id: "TASK-003"
task_revision: 1
mode: "implement"            # implement | fix
status: "implemented"         # implemented | blocked | failed
review_ready: true
source_spec: "tasks/cadastro-despesas/tasks/TASK-003.md"
workspace:
  isolation: "worktree"       # worktree | shared | unknown
  branch: null                # branch real ou null; não inventar
  commit: null                # hash real ou null; não gerar commit por padrão
changes:
  - path: "src/main/.../api/ExpenseHandler.java"
    purpose: "Expor endpoint com validação"
    within_scope: true
acceptance:
  - id: "AC-001"
    status: "verified"       # verified | partial | not_verified
    evidence: "Teste ... retornou 400 para entrada inválida"
verification:
  - id: "VER-001"
    status: "passed"         # passed | failed | not_run
    command: "./mvnw -q -Dtest=ExpenseHandlerTest test"
    evidence: "Teste existente, execução local: 1 passado"
    reason_if_not_run: null
    artifacts: []            # e2e de interface: caminhos das screenshots em results/TASK-###/screenshots/
  - id: "VER-020"
    status: "passed"
    command: "npm run test:e2e -- --grep VER-020"
    evidence: "Playwright: 1 passed; 3 capturas"
    reason_if_not_run: null
    artifacts:
      - "tasks/cadastro-despesas/results/TASK-003/screenshots/VER-020-01-lista-vazia.png"
      - "tasks/cadastro-despesas/results/TASK-003/screenshots/VER-020-02-formulario-preenchido.png"
      - "tasks/cadastro-despesas/results/TASK-003/screenshots/VER-020-03-despesa-criada.png"
review_findings: []           # no modo fix: FIND-###, estado e evidência
blockers: []
risks: []
next_step: "review"          # review | decision | retry
```

**Use dados reais:** `command` nulo quando não foi executado; `status: not_run` exige `reason_if_not_run`. Se não existe `commit` ou branch identificável, use `null`. Se a alteração não respeitou o escopo, bloqueie e corrija antes do handoff.

## `result.md` — conteúdo mínimo

```markdown
# Resultado — TASK-003 (revision 1)

## Resumo e escopo
[O que mudou, e por quê]

## Arquivos alterados
| Caminho | Motivo | Dentro do escopo? |
| --- | --- | --- |
| ... | ... | sim |

## Aceite e validação
| ID | Resultado | Evidência observável |
| --- | --- | --- |
| AC-001 | verificado / parcial / não verificado | ... |
| VER-001 | passou / falhou / não executou | ... |

## Comandos executados
| Comando | Resultado | Contexto |
| --- | --- | --- |
| ... | passou/falhou | ... |

## Review/fix (quando houver)
- Rodada: ...
- FIND-###: resolvido / não resolvido; evidência ...

## Riscos, blockers e próximos passos
...
```

Não reescreva o histórico para parecer que uma rodada antiga foi aprovada. Ao atualizar `result.md` em modo fix, acrescente uma seção de rodada com o estado atual e evidências novas. `result.yaml` representa o estado **mais recente**.

## Regras de consistência

- `task_id` e `task_revision` devem corresponder à spec e ao registro.
- `status: implemented` representa trabalho entregue ao coordenador **para review**, não trabalho integrado/aprovado.
- `review_ready: true` só se não restarem blockers materiais, e houver evidência suficiente para revisão.
- `status: blocked` implica `review_ready: false`, `blockers` com motivo objetivo e `next_step: decision` ou `retry`.
- `status: failed` descreve falha técnica ou verificação crítica não resolvida; não declarar sucesso.
- Verificações não executadas não podem ser declaradas aprovadas.
- `artifacts` lista só arquivos que existem no caminho indicado. Um `VER-*` e2e de interface `passed` sem as screenshots previstas no contrato é inconsistente: corrija ou marque `partial` no AC correspondente.
- `review_findings` podem estar vazios na primeira implementação; no modo fix incluem IDs reais do review, por exemplo:

```yaml
review_findings:
  - id: "FIND-001"
    status: "fixed"         # fixed | unresolved
    evidence: "Adicionada validação X; teste Y passou"
```

- Se especificação/revisão mudou enquanto worker executava, reporte incompatibilidade; não marque resultado antigo como novo.
