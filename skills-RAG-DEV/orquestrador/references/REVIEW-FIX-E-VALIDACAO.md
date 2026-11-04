# Ciclo review/fix e validação da história

## Review por task

```text
/task-review tasks/<slug>/tasks/TASK-001.md --round 01
```

Se `changes_requested`:

```text
/implementation tasks/<slug>/tasks/TASK-001.md --fix tasks/<slug>/reviews/TASK-001/round-01.md
/task-review tasks/<slug>/tasks/TASK-001.md --round 02
```

**Condições:** diff realmente disponível, `result.yaml` com a revisão correspondente, evidência dos testes e, quando possível, reviewer diferente do implementador. Os findings `FIND-###` são estáveis **por task**: a rodada seguinte reaproveita o ID de cada achado que ainda existir.

## Validação da história

Quando as tasks relevantes estiverem `done` e o código completo estiver integrado:

```text
/integration-validation tasks/<slug> --mode validate
```

O validador registra `accepted`, `changes_requested` ou `blocked` em `validation/integration-report.yaml` e descreve `REQ-* -> AC-* -> VER-* -> TASK-*` em `integration-report.md`.

## Tratando `VAL-###`

Achados `VAL-*` **não podem ser passados diretamente** para `--fix`, que exige um `round-NN.md` da Task Review.

- **Falha atribuível a uma task:** passe o relatório como contexto de uma nova rodada de review dessa task, por exemplo:
  `/task-review tasks/<slug>/tasks/TASK-001.md --round 02 --context tasks/<slug>/validation/integration-report.md`.
  O reviewer gera os `FIND-*` e o Fixer os corrige. A task sai de `done` e volta a `in_review`; o número da rodada continua a sequência da task (ver "Reabertura por `VAL-*`" em `MAQUINA-ESTADOS-E-DAG.md`). As rodadas de reabertura contam para o limite de 3 da task.
- **Falha entre tasks ou mudança de contrato:** reavalie a decomposição/planejamento e crie ou revise uma task específica, com aprovação. Não amplie o escopo da task existente silenciosamente.
- **Ambiente ou autorização ausente:** `blocked`. Não fabrique resultado de teste.

Depois das correções: nova integração, revisão do código afetado e **nova execução** de `integration-validation`. A skill guarda snapshots `integration-report-run-NN.*` antes de substituir o relatório canônico.
