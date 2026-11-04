# Contrato da planning para a skill de criação de tasks — schema v2

Arquivo obrigatório: `tasks/<slug>/planning-contract.yaml`.

## Fontes de verdade

| Conteúdo | Fonte de verdade | Quem escreve |
| --- | --- | --- |
| Stories (`REQ-*`), critérios e bordas (`AC-*`), regras `RN-*` | `user-stories.md` + `prd.md` (ou `plan.md` § 4 quando não houver PRD) | `/prd` (ou `/planning` sem PRD) |
| `REQ-*`/`AC-*` técnicos acrescentados pela planning (com PRD) | `user-stories.md`, seção `## AC técnicos (planning)` | `/planning` |
| Componentes (`CMP-*`), interfaces, decisões técnicas | `plan.md` + `adrs/` | `/planning` |
| Verificações (`VER-*`) | `tests.md` | `/planning` |
| Projeção estruturada de tudo acima | `planning-contract.yaml` | `/planning` |

O contrato é uma **projeção compacta**: não repete justificativas, nunca traz conteúdo ausente das fontes e é o que a `task-breakdown` consome. Ela consulta as fontes para contexto, preservando os IDs.

**Reconciliação:** atualize primeiro a fonte e depois o contrato, na ordem `user-stories.md` → `plan.md`/`tests.md` → contrato.
- Ao achar divergência, compare e preserve decisões humanas.
- Conflito de intenção: registre a dúvida e marque `blocked` até resolver.
- Correção inequívoca: sincronize e relate.
- Só entregue `ready` depois de conferir equivalência semântica. YAML válido não comprova isso.

**Origem:** `story.source` aponta para a fonte original. `historia.md` contém o texto original ou uma referência resolvível; o consumidor lê o conteúdo referenciado.

## Exemplo válido de formato

```yaml
schema_version: 2
story:
  id: "HIS-001"            # opcional: usar ID real; omitir quando não existir
  slug: "cadastro-de-despesas"
  source: "tasks/cadastro-de-despesas/historia.md"
  kind: "feature"           # feature | bug | improvement | requirement
  complexity: "standard"    # light | standard | deep
  status: "ready"           # ready | blocked
  summary: "Permitir o cadastro de despesas"
  blockers: []
  artifacts:
    prd: "tasks/cadastro-de-despesas/prd.md"                    # null quando não houver PRD
    user_stories: "tasks/cadastro-de-despesas/user-stories.md"  # null quando não houver PRD
    plan: "tasks/cadastro-de-despesas/plan.md"
    tests: "tasks/cadastro-de-despesas/tests.md"
    adrs_dir: "tasks/cadastro-de-despesas/adrs/"

decisions:
  - id: "ADR-001"
    scope: "product"        # product | technical
    title: "Despesa sempre vinculada a um centro de custo"
    path: "tasks/cadastro-de-despesas/adrs/ADR-001.md"
  - id: "ADR-002"
    scope: "technical"
    title: "Reutilizar ExpenseRepository existente em vez de novo agregado"
    path: "tasks/cadastro-de-despesas/adrs/ADR-002.md"

requirements:
  - id: "REQ-001"
    status: "active"        # active | withdrawn
    description: "Colaborador cadastra uma despesa com campos obrigatórios"
    persona: "Colaborador"  # obrigatório quando houver PRD; opcional sem PRD
    business_rule_ids: ["RN-001", "RN-002"]   # [] quando não houver
    acceptance_criteria:
      - id: "AC-001"
        status: "active"
        kind: "criterion"   # criterion | edge_case
        edge_class: null    # obrigatório em edge_case; ver lista abaixo
        description: "Despesa válida é persistida e o identificador é retornado"
        given: "colaborador autenticado com centro de custo ativo"
        when: "envia despesa com valor, data e categoria válidos"
        then: "a despesa é salva e o identificador aparece na resposta"
      - id: "AC-002"
        status: "active"
        kind: "edge_case"
        edge_class: "invalid_input"
        description: "Despesa sem valor é rejeitada"
        given: "colaborador autenticado"
        when: "envia despesa sem o campo valor"
        then: "recebe erro de validação indicando o campo valor e nada é salvo"

components:
  - id: "CMP-001"
    name: "Persistência"
    responsibility: "Salvar despesas e garantir integridade"
    requirement_ids: ["REQ-001"]
    acceptance_criteria_ids: ["AC-001"]
    impact: "modified"      # new | modified | deprecated
    depends_on: []
    candidate_paths: ["src/main/.../repository/"]
    path_confidence: "probable"   # confirmed | probable | unknown
    interfaces: ["ExpenseRepository.save(Expense): Mono<ExpenseId>"]   # assinaturas do plan.md; [] em light
    contracts_to_stabilize: ["Modelo Expense"]
    concurrent_edit_risks: []
  - id: "CMP-002"
    name: "API"
    responsibility: "Validar entrada e expor endpoint"
    requirement_ids: ["REQ-001"]
    acceptance_criteria_ids: ["AC-001", "AC-002"]
    impact: "new"
    depends_on: ["CMP-001"]
    candidate_paths: ["src/main/.../api/"]
    path_confidence: "probable"
    interfaces: ["POST /despesas"]
    contracts_to_stabilize: ["DTO da despesa", "Códigos HTTP"]
    concurrent_edit_risks: ["DTO compartilhado"]

verification:
  - id: "VER-001"
    status: "active"
    requirement_ids: ["REQ-001"]
    acceptance_criteria_ids: ["AC-001"]
    component_ids: ["CMP-002", "CMP-001"]
    type: "integration"     # unit | integration | contract | e2e | load | security | manual
    case_class: "happy"     # happy | error | boundary | concurrency | idempotency | ordering | state
    target: "POST /despesas"   # função, rota ou comando (real ou candidato)
    expected_evidence: "POST /despesas com valor=150.00, data=2026-10-01, categoria=TRANSPORTE → 201 com id não vazio; registro presente no repositório"
  - id: "VER-002"
    status: "active"
    requirement_ids: ["REQ-001"]
    acceptance_criteria_ids: ["AC-002"]
    component_ids: ["CMP-002"]
    type: "contract"
    case_class: "error"
    target: "POST /despesas"
    expected_evidence: "POST /despesas sem valor → 400 com code=validation.required e field=valor; nenhum registro criado"
  - id: "VER-020"
    status: "active"
    requirement_ids: ["REQ-001"]
    acceptance_criteria_ids: ["AC-001"]
    component_ids: ["CMP-003"]
    type: "e2e"
    case_class: "happy"
    target: "Tela Despesas → Nova despesa (Playwright)"
    expected_evidence: "Usuário preenche valor, data e categoria, confirma e vê a despesa na lista com o valor formatado"
    screenshots: ["01-lista-vazia", "02-formulario-preenchido", "03-despesa-criada"]   # só em e2e de interface; um por estado verificado

constraints:
  out_of_scope: ["Alterar consultas existentes"]
  non_functional: []        # métricas somente quando fundamentadas
  security: []
  assumptions: []
  risks: []
  external_research: "not_required"   # performed | not_required | unavailable
```

**Valores de `edge_class`:** `invalid_input`, `empty`, `limits`, `permissions`, `concurrency`, `interruption`, `repetition`, `ordering`, `state`, `scale`.

## Regras do contrato

1. O exemplo é ilustrativo: **não copie** valores para uma demanda real. Listas vazias ficam `[]`.
2. **Campos obrigatórios:** `story.id` é opcional. São obrigatórios `slug`, `source`, `kind`, `complexity`, `status`, `summary`, `blockers` e `artifacts`; em `artifacts`, `prd`/`user_stories` são `null` quando não houver PRD.
3. `decisions`, `requirements`, `components`, `verification` e `constraints` são obrigatórios. Em `standard`/`deep`, `decisions` tem ao menos um ADR `technical`.
4. **IDs vindos do PRD são preservados.** A planning pode acrescentar `REQ-*`/`AC-*` técnicos continuando a numeração, gravados na seção `## AC técnicos (planning)` de `user-stories.md` (ou em `plan.md` § 4 sem PRD); o contrato nunca traz um AC ausente dessas fontes. IDs nunca são renumerados nem reutilizados: item removido recebe `status: withdrawn` e permanece no arquivo.
5. **AC:**
   - todo `AC-*` tem `kind`, `description`, `given`, `when` e `then`;
   - `edge_case` exige `edge_class`; `criterion` usa `edge_class: null`;
   - em `light` sem PRD, `given/when/then` podem ser curtos, mas não vazios.
6. Todo `REQ-*` ativo tem ao menos um `AC-*` ativo do tipo `criterion`. Se faltar critério e isso comprometer a solução, `status: blocked`.
7. **CMP:**
   - aponta para `REQ-*`/`AC-*` válidos;
   - cada `depends_on` aponta para um `CMP-*` existente, sem autorreferência nem ciclos;
   - `depends_on` é **dependência técnica**, não ordem de tarefas.
8. **VER:**
   - um `VER-*` = **um comportamento observável**;
   - `target` e `expected_evidence` concretos: entrada, saída ou erro esperados, sem placeholders nem "verificar tratamento de erro";
   - `case_class` obrigatório em `unit`, recomendado nos demais;
   - `screenshots` (lista de nomes em kebab-case) obrigatório em `type: e2e` de interface web, um por estado verificado; ausente nos demais tipos;
   - referências a `REQ-*`, `AC-*` e `CMP-*` existentes e coerentes (o AC pertence ao REQ citado).
9. **Cobertura em `ready`:**
   - todo `REQ-*` ativo tem `AC-*` e `CMP-*`;
   - todo `AC-*` ativo (critério **e** borda) tem ao menos um `VER-*` ativo;
   - todo `CMP-*` tem ao menos um `VER-*` com `case_class: error` ou justificativa registrada em `tests.md`.
10. `candidate_paths` aceita paths reais ou candidatos; `path_confidence` indica o grau de confirmação. `unknown` só sem acesso ao repositório.
11. `external_research` registra a decisão explicitamente. Nunca `performed` sem fontes citadas no `plan.md`.
12. **Blockers:**
    - em `blocked`, liste dúvidas objetivas em `story.blockers`;
    - se o PRD estiver `blocked`, o contrato também fica `blocked`.

## Contrato mínimo em `light`

Mesmo schema v2, sem inventar componentes:
- `artifacts.prd`/`user_stories` podem ser `null`;
- `decisions` pode ser `[]`;
- `interfaces`, `depends_on`, `contracts_to_stabilize`, `concurrent_edit_risks`, `non_functional` e `assumptions` podem ser `[]`;
- os VER mínimos são: regressão que reproduz o bug, caminho feliz e erro afetado. Cada um continua concreto;
- não divida itens só para aumentar a quantidade de IDs.

## Validação mínima antes do handoff

- YAML parseável (com parser já disponível no projeto); sem tags YAML especiais.
- IDs únicos com referências resolvidas; sem ciclos entre `CMP-*`.
- Enums dentro dos valores permitidos: `status`, `complexity`, `kind`, `ac.kind`, `edge_class`, `impact`, `type`, `case_class`.
- Equivalência semântica:
  - REQ/AC com `user-stories.md`;
  - CMP e decisões com `plan.md`/`adrs/`;
  - VER com `tests.md`.

  Não é exigida prosa idêntica.
- Regras de cobertura do item 9 atendidas em `ready`.
- Blockers vazios em `ready`; em `blocked`, a causa concreta listada.
- Fonte original acessível ou limitação registrada.
- Sem parser, confira estas regras por leitura e declare a ausência de checagem automatizada.

## Compatibilidade com `schema_version: 1`

Contratos v1 continuam aceitos pelas skills seguintes, que tratam os campos novos como ausentes:
- AC sem `kind` → `criterion`;
- VER sem `component_ids`, `case_class` ou `target` → desconhecidos;
- sem `artifacts` → só `plan.md`;
- sem `status` → `active`.

Ao reexecutar `/planning` sobre um contrato v1, migre para v2 preservando todos os IDs e informe a migração.

## Comportamento esperado da `task-breakdown`

1. Leia o contrato, `plan.md`, `tests.md`, `user-stories.md` (quando houver) e a história. Confira status e consistência antes de criar tasks.
2. Crie tasks com IDs próprios, critérios de done e dependências, refinando os componentes. **Não presuma correspondência 1:1 entre `CMP-*` e task.**
3. Preserve as referências a `REQ-*`, `AC-*`, `VER-*` e `ADR-*`.
4. Atribua cada `VER-*` ativo a **exatamente uma** task responsável. Outras tasks podem citá-lo como regressão.
5. Defina o paralelismo pelas dependências **e** pelos riscos de concorrência em arquivos.
6. Ignore itens `withdrawn`. Não inicie tasks de um planejamento `blocked`.
