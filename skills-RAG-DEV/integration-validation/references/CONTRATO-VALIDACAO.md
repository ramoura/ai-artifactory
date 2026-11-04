# Contrato de validação final — schema_version 1

## Arquivos

```
tasks/<slug>/validation/
├── integration-report.yaml
└── integration-report.md
```

Um único validador/orquestrador controla a escrita desses caminhos. A skill **não edita `tasks.yaml`** nem os resultados de outras tasks. O relatório não é o estado global da execução.

## Exemplo ilustrativo de `integration-report.yaml`

```yaml
schema_version: 1
story:
  slug: "cadastro-despesas"
  planning_contract: "tasks/cadastro-despesas/planning-contract.yaml"
  planning_schema_version: 2   # 1 para contratos legados
mode: "validate" # validate | integrate-and-validate
workspace:
  branch: null
  commit: null
  baseline_ref: null
  integrated_code_verified: true
  integration_authorized: false
integrated_tasks:
  - task_id: "TASK-001"
    revision: 1
    result_path: "tasks/cadastro-despesas/results/TASK-001/result.yaml"
    review_path: "tasks/cadastro-despesas/reviews/TASK-001/round-01.md"
    review_decision: "approve"  # approve | changes_requested | blocked | not_found
    code_present: true
    integration_ref: null
acceptance:
  - requirement_id: "REQ-001"
    criterion_id: "AC-001"
    task_ids: ["TASK-001"]
    verification_ids: ["VER-001"]
    status: "verified" # verified | failed | not_verified
    evidence: "Teste de integração X validou a resposta definida no contrato"
verification:
  - id: "VER-001"
    type: "integration"
    status: "passed" # passed | failed | not_run
    command: "./mvnw -q -Dtest=... test"
    evidence: "Teste X passou na integração"
    reason_if_not_run: null
    artifacts: []   # e2e de interface: screenshots em validation/screenshots/run-NN/
findings: []
blockers: []
risks: []
decision: "accepted" # accepted | changes_requested | blocked
human_approval_required: false
next_step: "pr_delivery" # pr_delivery | fix | decision
```

Preencha **apenas com informações reais** e inclua todos os `AC-*`/`VER-*` de escopo, não somente os exemplos. Use `null` se `branch`, `commit`, `integration_ref` ou `command` não forem conhecidos; não inventar hashes nem baseline. Se `code_present: false` para task necessária, o gate não pode ser `accepted`.

## Findings

Quando houver problemas, use `VAL-*` para a história:

```yaml
findings:
  - id: "VAL-001"
    severity: "high"  # critical | high | medium | low
    status: "open"     # open | resolved
    title: "Contrato incompatível entre API e persistência"
    evidence: "[path/teste/comportamento conferidos]"
    requirement_ids: ["REQ-001"]
    acceptance_criteria_ids: ["AC-001"]
    verification_ids: ["VER-001"]
    task_ids: ["TASK-001", "TASK-002"]
    cause_class: "cross_task" # task | cross_task | preexisting | environment | unclear
    expected_resolution: "[resultado verificável sem design arbitrário]"
```

`VAL-*` não substitui `FIND-*`: para chamar Implementation `--fix`, solicite à Task Review uma rodada específica da task com o relatório de validação como contexto, ou replaneje uma task cruzada.

## Regras de decisão

- `accepted`: código integrado comprovado, reviews aprovados e compatíveis com as revisões atuais, critérios e testes essenciais com evidência, sem problemas materiais.
- `changes_requested`: defeito corrigível identificado com evidência e encaminhamento possível.
- `blocked`: sem workspace integrado, tasks não revisadas, acesso/ambiente essencial ausente, autorização ou decisão indispensável não obtida.
- `human_approval_required`: refletir a política e o escopo real do projeto. Se `true`, `accepted` representa só o gate técnico, não autorização de deploy/merge.
- `next_step`: `pr_delivery` apenas com gate técnico `accepted`; `fix` para defeitos; `decision` para bloqueios ou mudança de escopo.
- `not_run` exige `reason_if_not_run`. Nenhuma execução indireta deve ser declarada própria.

## Execução novamente e trilha de auditoria

Em reexecução com único escritor, antes de atualizar o relatório canônico copie os arquivos anteriores, se existirem, para o próximo par livre `integration-report-run-NN.yaml/.md` sob `validation/`. Não sobrescreva snapshots; em caso de falha de cópia ou edição concorrente, pare e peça reconciliação. O canônico representa a avaliação mais recente (mesmo reprovada) e identifica o baseline; nunca reutilize evidência de baseline anterior após correções sem reexecutar o que for afetado.
