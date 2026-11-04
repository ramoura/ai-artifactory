# Guia do Orquestrador — Pipeline de desenvolvimento multiagente v2

Este guia especifica **como coordenar** as skills `prd`, `planning`, `task-breakdown`, `implementation`, `code-review` e `integration-validation` no **Claude Code CLI**, sem Compozy e sem serviços adicionais. O coordenador pode operar com uma única sessão (mais seguro), com múltiplas sessões/worktrees autorizadas ou com Agent Teams, se disponível.

**Papel do orquestrador:** decidir o que pode começar, delegar o trabalho, proteger arquivos compartilhados, integrar código, recolher evidências e controlar os gates. O orquestrador é empacotado como a skill `orquestrador` (`skills-RAG-DEV/orquestrador/`), mas **não é uma skill de trabalho como as outras cinco**: ela é exclusiva do agente principal, só o usuário a invoca (`disable-model-invocation: true`) e nunca é delegada a subagentes.

## 1. Arquitetura e limites

```text
História, requisito ou bug
        |
        v
/prd (features/requisitos) → prd.md + user-stories.md + adrs/
        |   (bugs e ajustes light pulam esta etapa)
        v
/planning → plan.md + tests.md + adrs/ + planning-contract.yaml (schema v2)
        |
        v
/task-breakdown → tasks.yaml + TASK-###.md + execution-plan.md
        |
        v
   ORQUESTRADOR (único escritor de tasks.yaml)
        |
        +-- grupo elegível A --> /implementation TASK-001 --+
        |                                              |
        +-- grupo elegível B --> /implementation TASK-002 --+--> coletar resultados/diffs
        |                                              |        e revisar cada task
        +-- grupo elegível C --> /implementation TASK-003 --+
                                                                |
                                                    /code-review TASK-###
                                                    /          |          \
                                               approve   changes_requested  blocked
                                                    |          |             |
                                                 integrar   /implementation  decisão
                                                    |          --fix -> review
                                                    v
                                             testes/gates + done
                                                    |
                              dependentes liberadas após código integrado
                                                    |
                                             história integrada
                                                    |
                                          /integration-validation
                                             /       |       \
                                         accepted  changes   blocked
                                             |     requested   |
                                       entrega/PR  novo ciclo decisão
                                       com aprovação  de correção
```

**Avisos:** comandos começados por `/` são invocações no prompt interativo do Claude Code; não são executáveis de Bash. `tasks.yaml` é um **registro local do projeto**, independente da lista interna do Agent Teams. O fluxo gráfico não cria automaticamente subagentes; eles precisam ser invocados/atribuídos pelo coordenador, dentro das permissões disponíveis.

## 2. Fontes de verdade e autoridade de escrita

| Objeto | Escritor autorizado | Consumidores |
| --- | --- | --- |
| `historia.md`, `prd.md`, `user-stories.md`, `adrs/` (produto) | PRD | Planning, todos os agentes, coordenador |
| `plan.md`, `tests.md`, `adrs/` (técnico), `planning-contract.yaml` | Planning | Breakdown, todos os agentes, coordenador |
| `tasks.yaml`, estado do DAG | **Orquestrador exclusivamente**, após geração inicial pelo Breakdown | Todos (somente leitura) |
| `tasks/TASK-###.md`, `execution-plan.md` | Task Breakdown | Implementer, reviewer, coordenador |
| Código e testes dentro de `scope.allowed_paths` | Implementer/Fixer atribuído | Reviewer, integrador, validador |
| `results/TASK-###/result.yaml` e `.md` | Implementer/Fixer da respectiva task | Reviewer, coordenador, validador |
| `reviews/TASK-###/round-NN.md` | Reviewer independente da respectiva rodada | Fixer, coordenador, validador |
| `validation/integration-report.yaml` e `.md` | Validador com escritor único | Orquestrador, revisores, responsáveis pela entrega |

Se agentes trabalharem em worktrees distintas, **arquivos não aparecem automaticamente na worktree do coordenador**: ele deve coletar resultados/diffs e disponibilizar o código correto para revisão. Não exponha relatórios de uma worktree a outra por suposição.

## 3. Como iniciar uma história

### 3.1 Planejar e decompor

No CLI interativo, na raiz do repo:

```text
/prd docs/historias/minha-historia.md      # features e requisitos
/planning tasks/minha-historia             # ou /planning <bug> direto, sem PRD
/task-breakdown tasks/minha-historia
```

Antes de implementar, inspecione:

- `prd.md` (quando houver): `Status: ready`, regras `RN-*` com valores exatos, fora de escopo decidido.
- `user-stories.md`: critérios Given/When/Then e casos de borda por classe.
- `planning-contract.yaml`: `schema_version: 2` (ou 1 legado), `story.status: ready`, `story.blockers: []` e IDs `REQ-*`/`AC-*`/`CMP-*`/`VER-*`/`ADR-*` íntegros.
- `tests.md`: matriz de cobertura sem buracos; cada `VER-*` concreto.
- `plan.md`: solução coerente, riscos, contratos, alternativas quando pertinentes.
- `tasks.yaml`: DAG acíclico, IDs `TASK-*` únicos, `revision` consistente, todas as tasks inicialmente `pending`, cada `VER-*` com exatamente uma task dona (`owned_verification_ids`).
- `execution-plan.md`: ordens possíveis, `scope.allowed_paths`, `scope.avoid_paths`, arquivos compartilhados, contratos e recursos de teste.

**Checkpoint de aprovação:** mostre ao responsável as tarefas elegíveis, contratos que podem ser estabilizados primeiro, capacidade de paralelismo, riscos e abordagem de integração. Espere autorização antes da primeira alteração de código. Caso `story.status` seja `blocked`, não inicie tasks.

### 3.2 Selecionar tasks prontas (DAG)

Uma task **somente pode passar a `ready`** quando:

1. Cada task em `depends_on` está `done` na versão atual do registro.
2. O código das predecessoras está realmente integrado e acessível no workspace da task sucessora.
3. Contratos prévios (`contract_preconditions`) estão satisfeitos, os paths foram confirmados e não há blockers.
4. A execução não conflita com edições simultâneas, migrações, schemas, arquivos de build, recursos de testes ou limites corporativos.

Não confunda **independência no DAG** com ausência de **conflito de edição**. Uma task pode não depender logicamente de outra, mas exigir execução serial por compartilhar `pom.xml`, DTOs, schema ou fixtures. Registre a razão quando serializar.

**Exemplo:**

```text
TASK-001: Contrato        ──┬──> TASK-002: API ──┐
                            └──> TASK-003: Cliente ├──> TASK-004: Integração
                                                    ┘
```

Após `TASK-001` ser revisada, integrada e marcada `done`, `TASK-002` e `TASK-003` podem rodar juntas **se** seus arquivos/recursos forem isoláveis. `TASK-004` só começa após ambas estarem `done` e integradas.

## 4. Máquina de estados e transições

Apenas o **orquestrador** altera `tasks.yaml` após o Breakdown. Estados definidos pelo contrato v2:

| Estado | Significado | Próxima ação típica |
| --- | --- | --- |
| `pending` | Ainda não liberada | Avaliar dependências e escopo |
| `ready` | Elegível e autorizada | Atribuir implementador |
| `in_progress` | Implementação/correção em curso | Coletar resultado exclusivo |
| `in_review` | `implemented`, evidências e diff disponíveis | Acionar reviewer independente |
| `needs_fix` | Review pediu correções | Delegar `implementation --fix` |
| `integrated` | Mudança integrada; gate final da task pendente | Conferir testes/review pós-integração |
| `done` | Review aprovada + integração confirmada + gates essenciais | Desbloquear sucessoras |
| `blocked` | Falta dado, dependência, permissão ou decisão | Resolver condição, nunca declarar sucesso |

Não pule `in_review`/integração por causa de `status: implemented` em `result.yaml`. `approve` da revisão **não** equivale a `done`; `accepted` da validação global **não** equivale a autorização de deploy/merge.

Uma transição prática:

```text
pending -> ready -> in_progress -> in_review
                                     |     \
                           changes_requested approve
                                     |          \
                                 needs_fix      integrated -> done
                                     |
                                 in_progress -> in_review
```

**Regra de revisão:** limite de até **3 rodadas** por task sem autorização adicional. Se ainda houver defeito material, marque `blocked` e informe o que precisa ser decidido; não reinicie infinitamente. Cada correção precisa produzir nova evidência e nova review. Quando a integração alterar código (por exemplo, resolução de conflito), o delta alterado deve ser revisado novamente, dentro da governança aplicável.

## 5. Execução de uma task (handoff detalhado)

1. **Coordenador:** define a task `ready` e a atribui, incluindo branch/worktree autorizada e ownership de arquivos; depois marca `in_progress`.
2. **Implementador:** executa `/implementation tasks/<slug>/tasks/TASK-###.md` e retorna `result.yaml` e `result.md`, além dos arquivos de código/teste alterados.
3. **Coordenador:** valida `task_id`, `task_revision`, paths e evidências. Se `status: blocked` ou `failed`, não envia para review como se estivesse pronta. Se `status: implemented` e `review_ready: true`, torna o diff real acessível ao reviewer e marca `in_review`.
4. **Reviewer:** executa `/code-review tasks/<slug>/tasks/TASK-###.md --round 01`; escreve `round-01.md`, decisão `approve | changes_requested | blocked` e findings `FIND-*`.
5. **Se houver findings:** coordenador marca `needs_fix` e atribui `/implementation <TASK> --fix <round-NN.md>` a um agente, de preferência diferente do reviewer. O agente reporta o estado de cada `FIND-*`; o reviewer **confirma no código** e faz `--round 02` ou `03`.
6. **Integração:** após review aprovada, o coordenador agrega a implementação na branch/workspace autorizado, confere o diff integrado, executa gates apropriados e verifica se a revisão ainda cobre o código resultante.
7. **Conclusão:** só marca `done` quando código integrado, verificação essencial e aprovação de revisão forem atuais e comprovados.

**Registro recomendado do handoff na comunicação do coordenador:** task ID + revision, base/branch/worktree, paths permitidos/proibidos, dependências resolvidas, onde gravar resultado, diff/commit de referência, comando de teste autorizado e o próximo gate. Não inventar branch, commit, hash ou comando de teste.

## 6. Paralelismo sem nova instalação

### Modo A — sequencial (padrão mais previsível)

Uma sessão do Claude Code executa uma task de cada vez, acionando o review em contexto independente se disponível. Ideal para testar os contratos, corrigir bugs localizados e projetos com arquivos compartilhados. O orquestrador atualiza `tasks.yaml` ao final de cada gate.

### Modo B — múltiplas sessões/worktrees autorizadas

Se a empresa já permite Git worktrees e múltiplas sessões Claude Code, escolha até três tasks independentes e garanta ownership dos arquivos. Exemplo **ilustrativo, apenas se autorizado**:

```bash
git worktree add ../repo-task-002 -b feature/task-002
git worktree add ../repo-task-003 -b feature/task-003
```

Execute uma sessão CLI em cada diretório, forneça a spec correspondente e mantenha **somente o coordenador** como escritor do registro canônico. Ao término, colete os commits/diffs e artefatos de cada task com a estratégia corporativa aprovada. Evite edição conjunta de `tasks.yaml`, `pom.xml`, migrations e outros caminhos compartilhados. Um worktree **não** sincroniza automaticamente as mudanças de outro.

> Observação operacional: o Git geralmente compartilha a pasta `.git` entre worktrees, mas os diretórios de trabalho e seus arquivos são separados. Cada agente ainda precisa da spec da task e da versão atual dos contratos em seu workspace.

### Modo C — Agent Teams (se disponível e permitido)

O recurso é **experimental e desligado por padrão**. Em shell que permita variável de ambiente, a ativação opcional pode ser:

```bash
CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 claude
```

Conforme as políticas locais, a variável também pode ser configurada em `settings.json`; não tente contornar bloqueios corporativos. Peça ao coordenador que mantenha no máximo 3 implementadores e controle a correspondência entre tasks internas da equipe e IDs `TASK-*` do arquivo `tasks.yaml`. As tasks internas **não substituem o registro da pipeline**. Para trabalhos fortemente sequenciais ou com muito conflito, mantenha o modo A.

Documentação oficial: https://code.claude.com/docs/en/agent-teams

## 7. Review/fix e findings de integração

### Review por task

```text
/code-review tasks/minha-feature/tasks/TASK-001.md --round 01
```

Se `changes_requested`:

```text
/implementation tasks/minha-feature/tasks/TASK-001.md --fix tasks/minha-feature/reviews/TASK-001/round-01.md
/code-review tasks/minha-feature/tasks/TASK-001.md --round 02
```

**Condições:** diff efetivamente disponível, `result.yaml` com revisão correspondente, evidência dos testes e reviewer diferente do implementador quando possível. Findings são `FIND-###` estáveis **por task**; a próxima rodada reaproveita o ID de cada achado ainda existente.

### Validation por história

Quando as tasks relevantes estiverem `done` e o código completo integrado:

```text
/integration-validation tasks/minha-feature --mode validate
```

O validador registra `accepted`, `changes_requested` ou `blocked` em `validation/integration-report.yaml` e descreve `REQ-* -> AC-* -> VER-* -> TASK-*` em `integration-report.md`.

Em `changes_requested`, achados `VAL-*` **não podem ser passados diretamente** como parâmetro `--fix`, que exige um `round-NN.md` da Code Review:

- **Falha atribuível a uma task:** encaminhe o relatório como contexto para uma nova rodada de review dessa task, por exemplo `/code-review tasks/<slug>/tasks/TASK-001.md --round 02 --context tasks/<slug>/validation/integration-report.md`. O reviewer produz findings `FIND-*` e o Fixer os corrige.
- **Falha entre tasks ou mudança de contrato:** reavalie a decomposição/planejamento; crie/revise task específica sob aprovação. Não expandir escopo da task existente silenciosamente.
- **Ambiente ou autorização ausente:** `blocked`; não fabricar um resultado de teste.

Depois das correções, faça nova integração, revisão do código afetado e **nova execução de integration-validation**. A skill preserva snapshots `integration-report-run-NN.*` antes de substituir o relatório canônico.

## 8. Iniciar o orquestrador — agente principal

Na **sessão principal** do Claude Code, depois de `/prd` (se aplicável), `/planning` e `/task-breakdown`:

```text
/orquestrador <slug>
```

A skill `orquestrador` contém o protocolo completo (precondições, despacho, quality gates, restrições e saída final). Ela fica em `skills-RAG-DEV/orquestrador/SKILL.md`, com os detalhes em `references/`. **Essa skill é a fonte única do protocolo.** Altere-a lá, não neste guia.

Por que skill e não prompt colado:

- **Fonte única versionada**, junto com as demais skills da pipeline.
- **Slug passado como argumento**, sem edição manual do texto.
- **Detalhes carregados sob demanda**, a partir de `references/`.

Salvaguardas da skill:

- `disable-model-invocation: true`: só o usuário invoca; nenhum subagente vira um segundo orquestrador.
- **Sem `context: fork`**: roda no agente principal, que é quem coordena os subagentes.
- Se for carregada fora do agente principal, a própria skill manda parar.

**Dica:** as restrições permanentes (sem push/deploy, não inventar evidência) podem ficar também no `CLAUDE.md` local do projeto, quando autorizado, para valer fora da orquestração.

## 9. Checklist do coordenador

### Antes da execução

- [ ] Contratos Planning v2 válidos e `story.status: ready`, sem blockers.
- [ ] DAG válido e `TASK-*` com revision/scope/AC/VER reais.
- [ ] Riscos de concorrência, ownership e estratégia de integração declarados.
- [ ] Permissões e aprovação para modificar código confirmadas.
- [ ] Nenhuma task dependente liberada prematuramente.

### Para cada task

- [ ] `result.yaml`/`.md` existem e combinam com `task_id`/`revision`.
- [ ] Comandos e testes têm resultados reais, sem sucesso presumido.
- [ ] Diff real entregue ao revisor, com baseline e escopo identificáveis.
- [ ] Review independente com decisão e findings acionáveis.
- [ ] Correções re-revisadas; até 3 rodadas ou aprovação adicional.
- [ ] Mudanças integradas e eventual delta de conflito re-revisado.
- [ ] Somente coordenador registrou `done` em `tasks.yaml`.

### Final da história

- [ ] Workspace contém toda a implementação integrada e auditável.
- [ ] Todas as tasks relevantes `done` e sem review obsoleta.
- [ ] `integration-report.yaml` e `.md` atuais e rastreáveis.
- [ ] Todos os `REQ-*`, `AC-*`, `VER-*` cobertos e evidenciados.
- [ ] Achados `VAL-*` tratados e gate reexecutado após correções.
- [ ] Decisão técnica `accepted`, ou `changes_requested`/`blocked` com motivo.
- [ ] Aprovações humanas e restrições de PR/merge/deploy respeitadas.

## 10. Troubleshooting e decisões comuns

| Sintoma | Causa possível | Ação correta |
| --- | --- | --- |
| Skill não aparece no `/` | Pasta ou frontmatter incorreto; restrição do ambiente | Verificar `.claude/skills/<nome>/SKILL.md`, reiniciar sessão se necessário e conferir políticas |
| `/code-review` chama algo inesperado | Existe skill nativa com mesmo nome | Skill de projeto substitui nativa em `/code-review`; `/review` continua nativa. Se preferir, renomear a customizada para `review-task` e ajustar todos os comandos |
| A task sucessora falha mesmo com predecessora `done` | Código ainda está em outra worktree/branch | Integrar/disponibilizar mudança no workspace do sucessor; nunca liberar por status isolado |
| Reviewer emite `blocked` por ausência de diff | Resultado textual existe, mas código não está acessível | Reunir diff correto da revisão, em workspace autorizado |
| Dois agentes tocaram o mesmo arquivo | Ownership/isolamento insuficiente | Suspender a concorrência, reconciliar mudanças e revisar o delta integrado |
| `result.yaml` diz `implemented` mas testes falham | Implementação não concluiu gates | Registrar falha, delegar correção ou bloqueio; não marcar `done` |
| `VAL-###` veio de incompatibilidade entre tasks | Contratos divergentes | Nova task ou replanejamento; não passar VAL diretamente para `--fix` |
| Os agentes não têm acesso a worktree | Política/ambiente restrito | Modo sequencial com um coordenador, mantendo os mesmos artefatos |
| Agent Teams não está disponível | Feature experimental ou política corporativa | Usar modo A, ou sessões separadas autorizadas; não contornar controles da empresa |
| Review chegou à terceira rodada e continua reprovada | Problema persistente ou mal especificado | `blocked`, síntese de evidências e decisão humana/replanejamento |

## 11. Critérios de aceitação deste próprio workflow

O fluxo está funcionando como pretendido quando um teste controlado demonstra: (a) pelo menos duas tasks independentes com escopos sem conflito; (b) uma task sucessora bloqueada até a integração real de suas predecessoras; (c) um `FIND-*` que dispara `implementation --fix` e nova review; (d) execução global que verifica todos `AC-*` e `VER-*` no código integrado; e (e) nenhuma falsa transição `implemented -> done` nem merge/deploy automático.

**Escopo documental:** este guia foi consolidado a partir dos contratos v2 de fato entregues nas skills da pipeline. Ele não representa uma execução de ponta a ponta já testada no ambiente da empresa.

## Referências oficiais do Claude Code

- Skills e precedência de nomes: https://code.claude.com/docs/en/skills
- Agent Teams (experimental): https://code.claude.com/docs/en/agent-teams
