# Pipeline de desenvolvimento multiagente — Skills v2

Pipeline de engenharia baseada em **skills do Claude Code CLI**, com PRD guiado por perguntas, especificação técnica rastreável, decomposição em DAG, implementação por task, revisão independente e validação integrada.

**Público-alvo:** equipes que já usam Claude Code CLI e Git e não podem instalar um orquestrador adicional. **Orquestração:** realizada pelo agente principal do Claude Code (ou por um operador humano), conforme [GUIA-ORQUESTRADOR.md](GUIA-ORQUESTRADOR.md).

> **Importante:** as skills definem contratos e procedimentos, mas **não iniciam automaticamente** um scheduler persistente ou sincronização entre CLIs. O orquestrador é responsável por delegar tarefas, integrar branches/worktrees, transicionar estados e verificar gates. Compatibilidade com Devin CLI é possível por handoff manual de arquivos; não existe integração automática incluída.

## Conteúdo

| Skill | Responsabilidade | Entrada | Saída principal |
| --- | --- | --- | --- |
| `prd` | Descobre o quê/por quê/para quem com interrogatório, regras de negócio e casos de borda (features/requisitos) | História/arquivo/texto | `prd.md` + `user-stories.md` + `adrs/` |
| `planning` | Tech-spec: desenho técnico, interfaces, ADRs e contrato de testes | `tasks/<slug>` com PRD, ou história/bug direto | `plan.md` + `tests.md` + `planning-contract.yaml` (v2) |
| `task-breakdown` | Cria tasks pequenas, rastreáveis e DAG de dependências | `planning-contract.yaml` + `plan.md` + `tests.md` | `tasks.yaml`, `tasks/TASK-###.md`, `execution-plan.md` |
| `implementation` | Implementa **uma task** ou corrige findings sem ampliar o escopo | Spec `TASK-###.md`, opcionalmente review | Código/testes + `results/TASK-###/result.yaml` e `.md` |
| `code-review` | Revisa **uma task** de modo independente | Spec, resultado e diff real | `reviews/TASK-###/round-NN.md` com `FIND-*` |
| `integration-validation` | Valida a **história inteira** no código integrado | Plano, DAG, resultados, reviews e código integrado | `validation/integration-report.yaml` e `.md`, com `VAL-*` |
| `orquestrador` | Coordena o DAG no agente principal (só invocação manual) | `<slug>` | Estados em `tasks.yaml`, gates e relatório final |

Todas as saídas YAML desta pipeline usam contratos locais versionados (`planning-contract.yaml` em `schema_version: 2`, compatível com 1; os demais em `schema_version: 1`), **não** o formato interno de tasks do Claude Agent Teams.

## Instalação

As pastas abaixo devem estar na raiz do repositório (com os `references/` de cada skill preservados):

```text
repo/
├── .claude/
│   └── skills/
│       ├── prd/
│       │   ├── SKILL.md
│       │   └── references/
│       ├── planning/
│       │   ├── SKILL.md
│       │   └── references/
│       ├── task-breakdown/
│       │   ├── SKILL.md
│       │   └── references/
│       ├── implementation/
│       │   ├── SKILL.md
│       │   └── references/
│       ├── code-review/
│       │   ├── SKILL.md
│       │   └── references/
│       ├── integration-validation/
│       │   ├── SKILL.md
│       │   └── references/
│       └── orquestrador/
│           ├── SKILL.md
│           └── references/
├── README.md                       # este arquivo
└── GUIA-ORQUESTRADOR.md            # prompt e procedimento operacional
```

Use os pacotes v2 já preparados para cada skill. A presença de `SKILL.md` sozinha **não é suficiente** quando a skill referencia arquivos em `references/`.

Confira as políticas da empresa antes de rodar comandos, acessar rede, executar testes ou usar Git worktrees. Não é necessário instalar Compozy, Docker, MCP ou libs adicionais para o fluxo básico.

**Verificação inicial:** abra o Claude Code na raiz do repositório e confira, no menu `/`, se as skills estão disponíveis e se o CLI consegue ler seus arquivos de referência. O nome `code-review` tem uma particularidade: Claude Code possui uma skill nativa `/code-review`. Uma skill de projeto com o mesmo nome **substitui a nativa** nesse comando; a alias nativa `/review` continua sendo nativa. Se preferir não sobrescrevê-la, renomeie **tanto a pasta quanto o campo `name`** da sua skill para `review-task` e atualize os comandos deste guia.

Referência oficial sobre skills e prioridade de nomes: https://code.claude.com/docs/en/skills

## Início rápido (manual, sem Agent Teams)

Execute no prompt **interativo do Claude Code**, não no shell Bash:

```text
/prd docs/historias/cadastro-despesas.md     # features/requisitos; bugs vão direto para /planning
/planning tasks/cadastro-despesas
/task-breakdown tasks/cadastro-despesas
```

Inspecione e aprove o DAG/escopos. Depois, com a task liberada pelo coordenador:

```text
/implementation tasks/cadastro-despesas/tasks/TASK-001.md
/code-review tasks/cadastro-despesas/tasks/TASK-001.md --round 01
```

Se a revisão solicitar alterações:

```text
/implementation tasks/cadastro-despesas/tasks/TASK-001.md --fix tasks/cadastro-despesas/reviews/TASK-001/round-01.md
/code-review tasks/cadastro-despesas/tasks/TASK-001.md --round 02
```

Repita o ciclo apenas dentro do limite de **3 rodadas**, ou pare para decisão. Após todas as tasks estarem revisadas, integradas e marcadas `done` pelo coordenador:

```text
/integration-validation tasks/cadastro-despesas --mode validate
```

**O modo `validate` não integra branches.** Quando a empresa autorizar explicitamente a integração em workspace isolado, há também `--mode integrate-and-validate`. Nenhum dos modos autoriza por si só push, merge na principal ou deploy.

## Estrutura de artefatos por história

```text
tasks/<slug>/
├── historia.md                      # materializado pela /prd ou pela /planning
├── prd.md                           # /prd: visão, regras RN-*, fora de escopo, status
├── user-stories.md                  # /prd: REQ-* e AC-* (critérios e bordas)
├── adrs/ADR-###.md                  # decisões de produto e técnicas
├── plan.md                          # /planning: tech-spec
├── tests.md                         # /planning: VER-* concretos + matriz de cobertura
├── planning-contract.yaml           # schema v2: REQ / AC / CMP / VER / ADR; story.status
├── tasks.yaml                       # DAG e estado canônico (orquestrador escreve)
├── execution-plan.md
├── tasks/
│   ├── TASK-001.md
│   └── TASK-002.md
├── results/
│   ├── TASK-001/{result.yaml,result.md}
│   └── TASK-002/{result.yaml,result.md}
├── reviews/
│   ├── TASK-001/round-01.md
│   └── TASK-002/round-01.md
└── validation/
    ├── integration-report.yaml
    ├── integration-report.md
    └── integration-report-run-NN.*     # snapshots históricos, quando houver
```

**Identificadores:** `RN-*` (regra de negócio), `ADR-*` (decisão), `REQ-*` (requisito/story), `AC-*` (critério de aceite), `CMP-*` (componente), `VER-*` (verificação), `TASK-*` (tarefa), `FIND-*` (achado da review de task) e `VAL-*` (achado global de integração).

## Regras de segurança da execução

1. **Escritor único:** somente o orquestrador atualiza `tasks.yaml`. Implementadores, reviewers e validador escrevem apenas nos caminhos que lhes pertencem.
2. **Dependências:** task sucessora só começa quando todas as predecessoras estiverem `done` **e o código delas estiver acessível no workspace da sucessora**.
3. **Paralelismo:** no máximo três implementadores, apenas quando não houver dependências ou conflitos materiais de arquivos/recursos. Worktree evita colisão local, **não** garante merge sem conflitos.
4. **Evidência:** `result.yaml` com `implemented` não significa `done`. A transição exige review, testes essenciais e integração efetiva.
5. **Independência:** reviewer não deve ser o mesmo agente que implementou, quando houver alternativa. Uma review sem acesso ao diff real precisa ser `blocked`.
6. **Falhas:** achados de task são `FIND-*`, convertidos em correções via `implementation --fix`. Achados globais `VAL-*` devem passar por revisão contextual da task ou nova decomposição; não podem ser passados diretamente a `--fix`.
7. **Limites corporativos:** nenhuma alteração em produção, instalação, rede externa, migração destrutiva, push, merge em branch protegida ou deploy sem autorização específica.

## Execução com e sem paralelismo

- **Padrão recomendado:** comece sequencialmente para validar os contratos e os gates.
- **Paralelismo simples:** use várias sessões/worktrees do Claude Code **apenas se a empresa permitir**, com ownership claro e integração coordenada.
- **Agent Teams (opcional/experimental):** permite distribuir tarefas entre agentes, mas não substitui `tasks.yaml` nem os gates desta pipeline; é desabilitado por padrão e pode estar bloqueado por política corporativa. Consulte https://code.claude.com/docs/en/agent-teams.
- **Devin CLI (opcional):** pode ler a spec e o diff para produzir uma revisão, mas o output deverá ser convertido ao contrato exato de `round-NN.md` antes de ser usado pelo Fixer. Essa conversão não está automatizada neste pacote.

## Documentação operacional

Consulte **[GUIA-ORQUESTRADOR.md](GUIA-ORQUESTRADOR.md)** para o prompt completo do coordenador, matriz de estados, modo sequencial, paralelo, integração, tratamento de failures, modelo de handoff, checklist de encerramento e troubleshooting.

## Estado da validação do pacote

Os cinco conjuntos v2 fornecidos anteriormente foram examinados quanto aos **nomes de entrada/saída, caminhos, campos e decisões** e foram usados para compor esta documentação. **Não foram executados testes end-to-end reais** no Claude Code ou no Devin CLI do ambiente corporativo. Antes de adotar em produção de equipe, faça um piloto com duas tasks independentes, uma task dependente e pelo menos um ciclo de correção.
