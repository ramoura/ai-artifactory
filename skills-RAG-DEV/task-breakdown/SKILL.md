---
name: task-breakdown
description: Converte plan.md e planning-contract.yaml produzidos pela skill planning v2 em um backlog executável de tasks pequenas, rastreáveis, com DAG de dependências, isolamento de escopo e critérios verificáveis. Use depois de planning estar ready; não implementa código.
argument-hint: "[tasks/<slug> | tasks/<slug>/planning-contract.yaml]"
user-invocable: true
---

# Task Breakdown — decomposição rastreável e DAG (v2)

## Papel e limites

Você é um Tech Lead responsável **somente pela decomposição**, não pela implementação. Transforme um plano técnico aprovado em unidades de trabalho que possam ser atribuídas a agentes independentes, sem perder contexto ou criar dependências artificiais.

**Entrada:** `$ARGUMENTS` apontando para a pasta `tasks/<slug>/` ou para `planning-contract.yaml`. Se for informado o `plan.md`, resolva a pasta e encontre o contrato adjacente.

**Arquivos de leitura obrigatórios:**
- `tasks/<slug>/planning-contract.yaml`: contrato da Planning v2, com `schema_version: 2` (ou `1`, legado), `story`, `decisions`, `requirements`, `components`, `verification` e `constraints`.
- `tasks/<slug>/tests.md`: contrato de testes, com os casos `VER-*` concretos e a matriz de cobertura (v2).
- `tasks/<slug>/plan.md` — justificativas, interfaces, limites e estratégia de validação.
- `tasks/<slug>/user-stories.md` e `prd.md`, quando existirem: stories, regras `RN-*` e casos de borda, inclusive a seção `## AC técnicos (planning)`; servem de contexto, sem substituir o contrato.
- `tasks/<slug>/historia.md`, quando existir: fonte original para esclarecer escopo; não substitui o contrato.

**Saídas obrigatórias:**
- `tasks/<slug>/tasks.yaml` — registro de tasks e DAG; **fonte de verdade do grafo e estados**.
- `tasks/<slug>/tasks/TASK-###.md` — instruções autocontidas para cada implementador.
- `tasks/<slug>/execution-plan.md` — visão de ondas elegíveis, riscos de concorrência e gates.

**NÃO FAÇA:** editar código-fonte, executar uma task, invocar a skill de implementação automaticamente, instalar dependências, alterar o contrato da Planning v2, criar uma branch ou fazer commit/push. Não aprove um plano `blocked` para produção de tasks executáveis.

## Contratos de referência

Leia sob demanda:
- `references/CONTRATO-TASKS.md` — campos, estados e invariantes de `tasks.yaml` e `TASK-###.md`.
- `references/EXEMPLO-TASKS.yaml` — **somente exemplo**, nunca copiar paths, métricas ou regras de negócio.
- `references/TEMPLATE-TASK.md` — formato obrigatório do arquivo de tarefa.
- `references/REGRAS-DAG-E-CONCORRENCIA.md` — DAG, integração, worktrees e critérios de desbloqueio.

Respeite `CLAUDE.md`, `AGENTS.md` e políticas locais quando existentes. O contrato da planning define **o quê/por quê**; você define **como decompor** sem alterar a solução aprovada.

## Workflow

### 1. Resolver entrada e verificar pré-condições

1. Localize exatamente uma pasta `tasks/<slug>`. Sem pasta/contrato identificável, informe o problema; não invente um planejamento.
2. Leia os artefatos da Planning v2. Aceite `schema_version: 2` ou `1`. Em v1, trate os campos novos como ausentes, conforme a seção de compatibilidade do contrato da planning. Com versão desconhecida, interrompa e explique. Ignore itens com `status: withdrawn`.
3. **Bloqueie a decomposição executável** se `story.status != ready`, `story.blockers` não estiver vazio ou requisitos essenciais estiverem ambíguos. Registre pendências na resposta, sem fabricar tasks `ready`.
4. Valide IDs `REQ-*`, `AC-*`, `CMP-*`, `VER-*` e `ADR-*`: unicidade, referências existentes, cobertura de cada AC por VER (inclusive `edge_case`), dependências entre componentes válidas e ausência de ciclos. Se não houver interpretador YAML, faça checagem estruturada por leitura e declare o limite; não alegue validação por parser.
5. Compare o contrato ao `plan.md`, ao `tests.md` e ao `user-stories.md`; se houver divergências materiais, não altere silenciosamente os artefatos de origem: sinalize a inconsistência e pare.
6. Observe `constraints.out_of_scope`, requisitos não funcionais, riscos e regras de segurança. Não invente endpoints, SLO, frameworks, testes ou comandos inexistentes.

### 2. Definir fatias verticais e contratos

1. Divida por **entregas testáveis**, não por uma task por arquivo, camada ou componente.
2. Uma task pode mapear vários `CMP-*`; um `CMP-*` pode demandar várias tasks. Não assumir relação 1:1.
3. Prefira tasks independentes que entreguem comportamento observável; crie task de preparação de contrato/interface **somente** se outras tasks realmente precisarem de uma definição compartilhada para prosseguir.
4. Para mudanças simples (`light`), prefira 1–3 tasks, sem criar tarefas cerimoniais. Para `standard`/`deep`, decomponha o suficiente para limitar contexto, risco e conflitos; sem cotas artificiais.
5. Evite dividir tarefas que obrigariam dois agentes a editar o mesmo arquivo compartilhado ou a modificar simultaneamente o mesmo schema/migração.
6. Inclua migração, teste de contrato, documentação e rollout somente se forem necessários à entrega. Preservar a menor solução prevista no plano.

### 3. Projetar o DAG de dependências

Para cada task, determine:
- `depends_on`: **apenas** `TASK-*` que têm de estar integradas e verificadas antes do início desta.
- `scope.allowed_paths`: arquivos/diretórios que poderá modificar, com paths conhecidos ou candidatos explícitos.
- `scope.avoid_paths`: paths proibidos; respeitar escopo fora da mudança.
- `scope.shared_risks`: arquivos, schemas, fixtures ou recursos compartilhados que exigem exclusão mútua/serialização.
- `contract_preconditions`: interfaces que precisam estar estáveis antes do paralelismo.
- `requirement_ids`, `acceptance_criteria_ids`, `component_ids`, `verification_ids`: rastreabilidade aos IDs originais.
- `owned_verification_ids` (v2): os `VER-*` que **esta** task deve escrever e fazer passar. Cada `VER-*` ativo pertence a exatamente uma task; as demais podem citá-lo em `verification_ids` como regressão.
- `decision_ids` (v2): ADRs que a task precisa respeitar.

**Atenção:** `CMP-002 depends_on CMP-001` não obriga dependência entre todas as tasks desses componentes se o contrato entre eles já estiver estabilizado e os arquivos forem isolados. Quando não houver evidência suficiente, escolha dependência conservadora e explique.

**Não crie task de validação final da história.** O gate final é a skill `/integration-validation`, acionada pelo orquestrador quando todas as tasks estiverem `done` e integradas; ela não é uma unidade do DAG. Os `VER-*` de integração, contrato e E2E do `tests.md` são distribuídos entre as tasks de implementação que entregam o comportamento correspondente, cada um com exatamente uma dona.

### 4. Especificar cada task para um implementador com contexto isolado

Crie `TASK-001`, `TASK-002` etc. em ordem estável e sem reutilizar IDs para objetivos diferentes. Cada `tasks/TASK-###.md` deve conter:

- Objetivo e resultado esperado, sem ambiguidades.
- Origem e IDs de requisitos/aceite/componentes/verificações.
- `depends_on` e condição objetiva para começar.
- Contexto mínimo: caminhos reais, padrões encontrados no plano e links locais para o contrato.
- Contratos a preservar e mudanças explicitamente fora do escopo.
- Limites de edição e riscos de concorrência.
- Critérios de aceite do **trabalho**, com os `AC-*` em Given/When/Then (inclusive bordas), e verificações executáveis/factíveis, sem inventar comando.
- Os casos `VER-*` próprios **transcritos do `tests.md`** (alvo, entrada, saída esperada, classe), para o implementador escrever exatamente esses testes sem depender de contexto oculto.
- Evidência esperada, tratamento de erros, risco/rollback quando aplicável.
- Critérios para reportar `blocked` em vez de improvisar uma decisão de negócio.

**Uma task não deve assumir contexto oculto da conversa do agente que planejou.** Não copie o plano inteiro; aponte trechos e arquivos relevantes.

### 5. Construir o registro canônico

Produza `tasks.yaml` com `schema_version: 1`, referência à Planning v2, status inicial `pending` e todas as tasks em ordem estável, conforme `references/CONTRATO-TASKS.md`.

- `status: pending` é o estado inicial; **não** marcar como `done`, `ready` ou `in_progress` por ter criado arquivos.
- Somente o **orquestrador** atualiza estados no registro durante a execução. Implementadores e reviewers registram resultados em caminhos exclusivos por task.
- `depends_on` forma um DAG: sem autorreferências, IDs inexistentes, ciclos ou dependências de tasks posteriores por convenção.
- Em cada task, inclua `revision: 1` (ou preserve/incremente na reconciliação) para detectar resultados obsoletos.
- Preserve a cobertura agregada: cada `VER-*` ativo tem **exatamente uma** task em `owned_verification_ids`; todo `AC-*` ativo (critério e borda) tem pelo menos uma task; cada `REQ-*` é rastreável.

### 6. Criar o plano de execução e coordenação

Em `execution-plan.md`, registre:

- Tasks sem predecessores e próximas ondas **potenciais**, deixando claro que são uma previsão estática.
- Matriz de concorrência: combinações seguras, conflitos de paths, contratos ou recursos.
- Máximo de implementadores recomendado: **até 3**, salvo restrição ou capacidade menor do ambiente.
- Preferência por worktrees isoladas, quando disponíveis e permitidas. Se não houver worktrees, somente edição paralela comprovadamente disjunta no mesmo workspace; se incerto, serializar.
- Gates de execução: dependências `done` + artefatos integrados, contratos estáveis, testes pertinentes, review independente por task e, ao final, `/integration-validation` sobre o código integrado.
- Protocolo de handoff para `/implementation`, resultado por task, reviewer, fixer e integração, descrito na referência.

A **task de revisão** em si não precisa estar no DAG se for um gate de aprovação do orquestrador. O registro não substitui revisão humana nas mudanças sensíveis.

### 7. Reexecução idempotente / mudanças de plano

Se `tasks.yaml` já existir:

1. Compare tasks novas e antigas; **preserve IDs e revisões** das tasks cujo objetivo/contrato/escopo não mudou.
2. Não descarte progresso, resultados ou achados de review.
3. Se uma task mudou materialmente, incremente sua `revision`, registre diferenças e marque seu resultado anterior como **obsoleto** para revalidação, sem apagar o histórico.
4. Não altere automaticamente tasks `in_progress`, `in_review` ou `done`, nem dependências de tasks em execução. Peça autorização antes de replanejar trabalho em andamento.
5. Nunca sobrescreva `planning-contract.yaml` ou `plan.md` para adaptar um DAG problemático.

### 8. Validar e concluir

Confirme:

- [ ] Origem Planning v2 presente, consistente e `ready`, sem blockers.
- [ ] Nenhuma alteração de código-fonte, branch ou estado de produção.
- [ ] Todos `TASK-*` únicos, arquivos correspondentes e `revision` consistente.
- [ ] `depends_on` aponta para tasks existentes e não forma ciclos.
- [ ] Todo `REQ-*` e `AC-*` (inclusive bordas) coberto; cada `VER-*` com exatamente uma task dona; nenhuma task sem objetivo verificável.
- [ ] Sem paralelismo declarado para mudanças conflitantes; contratos prévios registrados.
- [ ] Gates de integração definidos proporcionalmente ao risco; nenhuma task `kind: validation` ou `kind: integration` no DAG (o gate final é `/integration-validation`).
- [ ] Paths confirmados vs. candidatos sinalizados; nenhuma regra ou métrica inventada.
- [ ] Reexecução preservou tarefas existentes e reportou alterações (se aplicável).

Relate: caminhos gerados, número de tasks, dependências críticas, grupos potencialmente paralelos, riscos e próximo passo. **Não inicie implementação sem solicitação**.
