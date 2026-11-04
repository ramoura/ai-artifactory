# PRD v2 (Claude Code)

Primeira etapa da pipeline para **features e requisitos**. Descobre o *o quê / por quê / para quem* antes da `/planning` desenhar o *como*.

Baseada nos pontos fortes de `cy-create-prd` (Compozy): interrogatório guiado por árvore de decisões, catálogo de user stories com Given/When/Then, varredura de casos de borda e ADRs. Foi adaptada para ambiente corporativo:
- pesquisa externa condicional e sem dados internos;
- proporcionalidade (bugs pulam o PRD);
- IDs compatíveis com o contrato da pipeline (`REQ-*`, `AC-*`).

## Instalação

```text
.claude/skills/prd/
├── SKILL.md
└── references/
    ├── PROTOCOLO-PERGUNTAS.md
    ├── TEMPLATE-PRD.md
    ├── TEMPLATE-USER-STORIES.md
    └── TEMPLATE-ADR.md
```

## Uso

```text
/prd docs/historias/cadastro-despesas.md
/prd Permitir que gestores aprovem despesas em lote
```

## Saídas

```text
tasks/<slug>/
├── historia.md
├── prd.md              # visão, objetivos, regras RN-*, UX, restrições, fora de escopo, status
├── user-stories.md     # REQ-* (stories) e AC-* (critérios e casos de borda)
└── adrs/ADR-###.md     # decisões de produto
```

Próximo passo: `/planning tasks/<slug>`. A `planning` preserva os IDs `REQ-*`/`AC-*` e acrescenta os componentes, as verificações e o contrato.
