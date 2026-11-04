# Planning v2 — tech-spec (Claude Code)

Segunda etapa da pipeline. Transforma o PRD (`/prd`), ou a história direta no caso de bugs e ajustes, em especificação técnica, contrato de testes e contrato estruturado para a `task-breakdown`.

Incorpora os pontos fortes de `cy-create-techspec` (Compozy):
- interrogatório técnico com recomendação;
- interfaces centrais em código;
- tabela de impacto;
- ADRs;
- `tests.md` com casos concretos e matriz de cobertura.

Mantém o que já era forte aqui: profundidade proporcional, evidência rastreável, nenhum NFR inventado, pesquisa externa condicional e o gate `ready`/`blocked`.

## Instalação

```text
.claude/skills/planning/
├── SKILL.md
└── references/
    ├── CONTRATO-PLANEJAMENTO.md   # schema v2 (compatível com v1)
    ├── CRITERIOS-ENGENHARIA.md
    ├── TEMPLATE-GENERICO.md
    ├── TEMPLATE-JAVA-WEBFLUX.md
    ├── TEMPLATE-TESTS.md
    └── TEMPLATE-ADR.md            # igual a prd/references/TEMPLATE-ADR.md
```

A `planning` também lê `../implementation/references/stacks/<stack>.md` (skill irmã) quando a stack é Spring MVC, Spring WebFlux, Node backend ou React. Instale as duas skills juntas; sem a `implementation`, a planning segue só com as convenções do repositório.

Se já existe uma skill `planning`, salve uma cópia antes de substituir. `TEMPLATE-ADR.md` é duplicado de propósito (cada skill é instalável isoladamente); mantenha as duas cópias iguais.

## Uso

```text
/planning tasks/cadastro-despesas                  # feature, depois de /prd
/planning Corrigir duplicidade no consumo de eventos # bug: direto, sem PRD
```

## Saídas

```text
tasks/<slug>/
├── historia.md               # criado aqui só quando não houver PRD
├── prd.md, user-stories.md   # lidos (da /prd), nunca alterados
├── plan.md                   # tech-spec
├── tests.md                  # VER-* concretos + matriz de cobertura
├── adrs/ADR-###.md           # decisões técnicas (numeração compartilhada com a /prd)
└── planning-contract.yaml    # schema_version: 2
```

O contrato é a entrada da `task-breakdown`. A `planning` não gera tasks nem implementa código.

## Atenção

Evolução **reconstruída a partir da gravação e da análise**, não uma cópia byte a byte da skill original. Revise contra seus templates e regras corporativas antes de adotá-la. As permissões corporativas continuam valendo.
