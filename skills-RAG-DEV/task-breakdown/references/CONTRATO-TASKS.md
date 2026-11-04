# Contrato entre Task Breakdown, orquestrador e Implementation

## Fonte e destino

**Input:** `tasks/<slug>/planning-contract.yaml` (`schema_version: 2`, ou `1` legado) + `plan.md` + `tests.md` (+ `user-stories.md` quando houver PRD), da Planning v2.

**Output:**
- `tasks/<slug>/tasks.yaml` — DAG e status canônicos; único escritor durante execução: orquestrador.
- `tasks/<slug>/tasks/TASK-###.md` — especificação; de preferência imutável durante execução, com revisão explícita se mudar.
- `tasks/<slug>/execution-plan.md` — ondas, conflitos e gates.

**Runtime da Implementation:** `tasks/<slug>/results/TASK-###/result.yaml` e `result.md` (um diretório por task). **Review:** `tasks/<slug>/reviews/TASK-###/round-01.md` etc. Em worktree, estes resultados só se tornam visíveis ao coordenador após integração/coleta; nunca pressupor sincronização instantânea.

## `tasks.yaml`: campos exigidos

```yaml
schema_version: 1
story:
  slug: "exemplo"
  planning_contract: "tasks/exemplo/planning-contract.yaml"
  plan: "tasks/exemplo/plan.md"
  planning_schema_version: 2   # 1 para contratos legados
  tests: "tasks/exemplo/tests.md"
  status_at_breakdown: "ready"

coordination:
  max_parallel_implementers: 3
  dependency_gate: "done_and_integrated"
  registry_writer: "orchestrator_only"
  review_round_limit: 3

tasks:
  - id: "TASK-001"
    revision: 1
    title: "Definir contrato de API"
    kind: "contract"   # contract | implementation | migration
    status: "pending" # pending | ready | in_progress | in_review | needs_fix | integrated | done | blocked
    specification: "tasks/exemplo/tasks/TASK-001.md"
    requirement_ids: ["REQ-001"]
    acceptance_criteria_ids: ["AC-001"]
    component_ids: ["CMP-001"]
    verification_ids: ["VER-001"]        # todas as VER relevantes (inclusive regressões de outras tasks)
    owned_verification_ids: ["VER-001"]  # v2: VER que esta task escreve; cada VER tem exatamente uma dona
    decision_ids: []                      # v2: ADR-* que a task deve respeitar
    depends_on: []
    contract_preconditions: []
    scope:
      allowed_paths: ["src/main/java/.../"]
      avoid_paths: []
      shared_risks: []
    readiness_notes: "Contrato estabilizado no escopo da própria task"
```

Substitua por dados reais, sem copiar placeholders.

## Estados e autoridade

| Estado | Significado | Quem determina |
| --- | --- | --- |
| `pending` | Registrada, ainda não liberada | Breakdown |
| `ready` | Dependências `done`, integração e isolamento satisfeitos | Orquestrador |
| `in_progress` | Implementador autorizado está trabalhando | Orquestrador |
| `in_review` | Implementação e evidências entregues; aguarda revisão | Orquestrador |
| `needs_fix` | Reviewer rejeitou com findings acionáveis | Orquestrador |
| `integrated` | Mudanças integradas, mas gate final pendente | Orquestrador |
| `done` | Implementação integrada, verificada e aprovada conforme gates | Orquestrador |
| `blocked` | Dependência, requisito, permissão ou falha impede progresso | Orquestrador |

`status` no registro pode estar desatualizado durante execução em worktrees; o coordenador deve reconciliar os resultados e a integração antes de liberar sucessoras. **Não tratar `implemented` em result.yaml como `done`**.

Regra principal: uma sucessora só pode começar quando **todos** os IDs de `depends_on` estiverem `done` **e** seus artefatos de código estiverem disponíveis no workspace da sucessora. Isso é mais rígido que apenas receber mensagem de conclusão de outro agente.

## Invariantes obrigatórias

1. IDs `TASK-*` únicos e estáveis; `revision` inteira positiva.
2. Todas as referências a `REQ`, `AC`, `CMP` e `VER` existentes no contrato da Planning v2.
3. Nenhuma task depende dela mesma, de ID inexistente ou de ciclo (verificação topológica).
4. Todo `AC-*` ativo (critério e borda) tem ao menos uma task. Todo `VER-*` ativo aparece em `owned_verification_ids` de **exatamente uma** task (v2; em v1, ao menos uma responsável). Itens `withdrawn` são ignorados.
5. Não declarar paths candidatos como confirmados. Um path desconhecido exige exploração controlada ou bloqueio antes de editar.
6. Tasks independentes no DAG **não são automaticamente seguras em paralelo**. É preciso verificar edição concorrente, ambientes, schemas, migrações e contratos.
7. O DAG não contém task de validação final. Os `VER-*` de integração, contrato e E2E pertencem às tasks de implementação; o gate final da história é a skill `/integration-validation`, acionada pelo orquestrador fora do DAG.
8. Adaptações de escopo ou dependências que alterem comportamento do plano exigem revisão da Planning, não decisão silenciosa da Breakdown.
9. Planejamento `blocked` não gera tasks executáveis.
10. Escritas concorrentes em `tasks.yaml` são proibidas. O registro pertence ao orquestrador.

## Handoff entre agentes

1. Breakdown gera registro e especificações (não executa).
2. Orquestrador escolhe tasks `ready` sem conflitos, despacha até capacidade autorizada.
3. Implementador executa `/implementation tasks/<slug>/tasks/TASK-###.md` no workspace autorizado.
4. Implementador grava `results/TASK-###/result.yaml` e `result.md` de forma isolada.
5. Orquestrador integra/coleta alterações, roda verificações pertinentes e encaminha para reviewer independente.
6. Reviewer grava `reviews/TASK-###/round-NN.md`; se rejeitar, orquestrador chama `/implementation <spec> --fix <review-path>` em no máximo 3 rodadas.
7. Só após integração, testes e aprovação, orquestrador marca `done` e desbloqueia dependentes.
8. Gate final por história: o orquestrador executa `/integration-validation`, que considera toda a matriz `REQ` → `AC` → `VER`, inclusive integração e regressão.

A habilidade de criar subagentes/teams é opcional e depende do Claude Code da empresa. Este contrato também serve para execução sequencial ou coordenação manual.
