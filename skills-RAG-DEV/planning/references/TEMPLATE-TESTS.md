# Template de `tasks/<slug>/tests.md`

Contrato canônico de testes da história. Todo caso tem um ID estável `VER-###` e é projetado no `planning-contract.yaml`.

Quem consome:
- a `task-breakdown`, que atribui cada `VER-*` a **exatamente uma** task responsável;
- o implementador, que escreve **exatamente** os casos atribuídos;
- a `task-review` e a `integration-validation`, que conferem a suíte entregue contra este documento.

Um comportamento sem `VER-*` aqui é um comportamento que ninguém se comprometeu a verificar.

## Regras de ID

- `VER-###`, sequencial e global na história. O nível vai no campo `type` (`unit`, `integration`, `contract`, `e2e`, `load`, `security`, `manual`).
- IDs são permanentes depois que as tasks os referenciam: nunca renumere nem reutilize. Caso descartado fica `(withdrawn)` no lugar.

## Esqueleto

```markdown
# Contrato de testes — [nome da feature]

Derivado de `user-stories.md` (comportamento) e `plan.md` (componentes).

## Estratégia
- Frameworks e harness: [os do repositório, com caminho de exemplo]
- Fakes/mocks: apenas nas fronteiras de I/O
- Execução: [comandos existentes para unit/integration/e2e, ou `não verificado`]
- Convenções: [nomenclatura, estilo table-driven, fixtures — conforme o repositório]

## Matriz de cobertura

| Origem | Comportamento | Unit | Integração/Contrato | E2E |
|---|---|---|---|---|
| REQ-001 / AC-001 | [critério] | VER-001 | VER-010 | VER-020 |
| REQ-001 / AC-003 (edge · permissions) | [borda] | VER-002 | — | — |
| CMP-001 | [responsabilidade] | VER-001–VER-004 | VER-010 | — |
| POST /despesas | [contrato] | — | VER-011, VER-012 | — |

## Unitários

### CMP-001 — [componente] (plan.md § [seção])
- **VER-001** (unit · happy) — AC-001 — `ExpenseService.create`: dado [entrada concreta], retorna [saída concreta].
- **VER-002** (unit · error) — AC-003 — `ExpenseService.create`: dado [usuário sem papel APPROVER], lança [`ForbiddenException` com code=`expense.forbidden`].
- **VER-003** (unit · boundary) — AC-005 — no valor exato [10.000,00], [aceita]; em [10.000,01], [rejeita com code=`expense.limit`].

## Integração e contrato

### [Fronteira ou fluxo]
- **VER-010** (integration) — AC-001 — setup [fixtures/estado]; ação [chamada real entre componentes]; espera [resultado observável na fronteira].
- **VER-011** (contract) — `POST /despesas` sem `valor` → `400` com `code=validation.required`, campo `valor`.

## End-to-end

### [Jornada] (REQ-001, REQ-003)
- **VER-020** (e2e · Playwright) — REQ-001 — [ponto de entrada] → [passos visíveis ao usuário] → [resultado final observável]. Telas a capturar: `01-lista-vazia`, `02-formulario-preenchido`, `03-despesa-criada`.
```

## Exigências de cobertura (gate de `ready`)

- Todo `AC-*` de `user-stories.md` (ou do `plan.md`, sem PRD), **critério ou borda**, tem sua linha na matriz com pelo menos um `VER-*`.
- Todo `CMP-*` tem linha com cobertura unitária **incluindo caminhos de erro**. Componente só com caso feliz conta como descoberto.
- Todo endpoint, comando ou mensagem do `plan.md` tem casos para a resposta de sucesso e para cada falha documentada.
- Toda jornada (`REQ-*`) tem pelo menos um caso E2E ou de integração do início ao fim.
- Célula vazia numa linha preenchida é aceitável. **Linha sem nenhum ID é um buraco:** preencha, ou anote o motivo de não precisar de teste.

## Regras de escrita dos casos

- **Concreto ou nada:** nomeie a função, rota ou comando real (ou candidato, marcado `probable`), a entrada exata e a saída ou erro esperados. "Verificar tratamento de erro" não é caso; "`POST /runs` com workflow inexistente → `404` com `code=workflow_not_found`" é.
- **Etiquete todo caso unitário com a classe:** `happy`, `error`, `boundary`, `concurrency`, `idempotency`, `ordering` ou `state`.
- **Um comportamento observável por caso.** Caso que precisa de dois "e" vira dois casos.
- **Níveis:** unitário faz fake só de I/O; integração usa a ligação real entre componentes; E2E passa pela superfície pública (API, CLI, UI) como o usuário.
- **Falhas em todos os níveis,** não só no unitário: fluxo interrompido, permissão negada e atores concorrentes espelham as classes de borda do PRD.
- **Sem metas inventadas:** cobertura percentual e carga só com fonte (política do projeto, SLO declarado).
- **E2E de interface usa Playwright** (ver `../../implementation/references/stacks/playwright-e2e.md`), salvo ferramenta já adotada pelo repositório. Todo caso e2e lista as **telas a capturar**, uma por estado verificado, com nome curto em kebab-case; esses nomes viram `screenshots` no contrato e os arquivos `VER-###-NN-<estado>.png` na evidência. Se o repositório não tem Playwright, registre como premissa ou blocker na § 9 do `plan.md`; a `implementation` não instala.
- **Proposto, não executado:** este documento descreve casos; nunca afirma que rodaram.

## Formato compacto (`light`)

- Estratégia em uma linha.
- Matriz só com os `AC-*` da correção.
- Os casos mínimos: **regressão que reproduz o bug** + caminho feliz + erro afetado.
- Continua concreto e com IDs `VER-*`.
