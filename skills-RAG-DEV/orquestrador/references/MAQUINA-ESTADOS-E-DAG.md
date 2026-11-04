# Máquina de estados, DAG e handoff

## Quando uma task pode passar a `ready`

Uma task **só pode passar a `ready`** quando:

1. Cada task em `depends_on` está `done` na versão atual do registro.
2. O código das predecessoras está realmente integrado e acessível no workspace da task sucessora.
3. Os contratos prévios (`contract_preconditions`) estão satisfeitos, os paths foram confirmados e não há blockers.
4. A execução não conflita com edições simultâneas, migrações, schemas, arquivos de build, recursos de teste ou limites corporativos.

Não confunda **independência no DAG** com ausência de **conflito de edição**. Uma task pode não depender logicamente de outra e ainda assim exigir execução serial por compartilhar `pom.xml`, DTOs, schema ou fixtures. Registre o motivo quando serializar.

```text
TASK-001: Contrato ──┬──> TASK-002: API     ──┐
                     └──> TASK-003: Cliente ──┴──> /integration-validation (fora do DAG)
```

Depois que `TASK-001` for revisada, integrada e marcada `done`, `TASK-002` e `TASK-003` podem rodar juntas **se** seus arquivos e recursos forem isoláveis. A `/integration-validation` só roda quando ambas estiverem `done` e integradas.

## Estados (contrato v2)

Só o orquestrador altera `tasks.yaml` depois do Breakdown.

| Estado | Significado | Próxima ação típica |
| --- | --- | --- |
| `pending` | Ainda não liberada | Avaliar dependências e escopo |
| `ready` | Elegível e autorizada | Atribuir implementador |
| `in_progress` | Implementação/correção em curso | Coletar resultado exclusivo |
| `in_review` | `implemented`, evidências e diff disponíveis | Acionar reviewer independente |
| `needs_fix` | Review pediu correções | Delegar `implementation --fix` |
| `integrated` | Mudança integrada; gate final da task pendente | Conferir testes/review pós-integração |
| `done` | Review aprovada + integração confirmada + gates essenciais | Desbloquear sucessoras |
| `blocked` | Falta dado, dependência, permissão ou decisão | Resolver a condição; nunca declarar sucesso |

Não pule `in_review` nem a integração só porque `result.yaml` diz `status: implemented`. `approve` da revisão **não** equivale a `done`.

```text
pending -> ready -> in_progress -> in_review
                                     |     \
                           changes_requested approve
                                     |          \
                                 needs_fix      integrated -> done
                                     |
                                 in_progress -> in_review
```

**Reabertura por `VAL-*`:** uma task `done` que receba achado da validação da história volta a `in_review` para a rodada com `--context`; se a review gerar `FIND-*`, segue para `needs_fix` e refaz o ciclo normal. A `revision` da spec não muda (o escopo é o mesmo); a rodada continua a numeração anterior (`round-02` se a última foi `round-01`). Tasks dependentes já `done` não são revertidas, mas a validação global precisa ser reexecutada depois da correção. Registre em `tasks.yaml` o motivo da reabertura.

```text
done --(VAL-* atribuível)--> in_review --(FIND-*)--> needs_fix --> ... --> done --> nova /integration-validation
```

**Regra de revisão:** no máximo **3 rodadas** por task sem autorização adicional. Se ainda houver defeito material, marque `blocked` e diga o que precisa ser decidido. Cada correção precisa gerar nova evidência e nova review. Quando a integração alterar código, o delta deve ser revisado de novo.

## Handoff de uma task

1. **Orquestrador:** marca a task `ready` e a atribui, informando branch/worktree autorizada e ownership de arquivos; depois marca `in_progress`.
2. **Implementador:** executa `/implementation tasks/<slug>/tasks/TASK-###.md` e devolve `result.yaml`, `result.md` e os arquivos de código/teste alterados.
3. **Orquestrador:** valida `task_id`, `task_revision`, paths e evidências. Se `status: blocked` ou `failed`, não envia para review como se estivesse pronta. Se `status: implemented` e `review_ready: true`, deixa o diff real acessível ao reviewer e marca `in_review`.
4. **Reviewer:** executa `/task-review tasks/<slug>/tasks/TASK-###.md --round 01` e grava `round-01.md` com a decisão `approve | changes_requested | blocked` e os findings `FIND-*`.
5. **Com findings:** o orquestrador marca `needs_fix` e atribui `/implementation <TASK> --fix <round-NN.md>`, de preferência a um agente diferente do reviewer. O reviewer **confirma no código** e faz `--round 02` ou `03`.
6. **Integração:** com a review aprovada, o orquestrador integra a mudança na branch/workspace autorizado, confere o diff integrado, roda os gates e verifica se a revisão ainda cobre o código resultante.
7. **Conclusão:** marca `done` só quando código integrado, verificação essencial e aprovação de revisão forem atuais e comprovados.

**Registro do handoff:** task ID + revision, base/branch/worktree, paths permitidos e proibidos, dependências resolvidas, onde gravar o resultado, diff/commit de referência, comando de teste autorizado e próximo gate. Não invente branch, commit, hash ou comando de teste.
