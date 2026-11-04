# Template obrigatório para `tasks/<slug>/tasks/TASK-###.md`

> Substitua placeholders somente por informações fundamentadas no plano e no repositório.

# TASK-### — [título]

## Identificação
- Revisão: 1
- Tipo: contract | implementation | migration
- História: `tasks/<slug>/historia.md` (se existir)
- Plano: `tasks/<slug>/plan.md`
- Contrato: `tasks/<slug>/planning-contract.yaml`
- Contrato de testes: `tasks/<slug>/tests.md`
- User stories: `tasks/<slug>/user-stories.md` (se existir)
- Decisões a respeitar: `ADR-*` / nenhuma
- Requisitos: `REQ-*`
- Aceites: `AC-*`
- Componentes: `CMP-*`
- Verificações próprias (esta task escreve): `VER-*`
- Verificações de regressão (de outras tasks): `VER-*` / nenhuma
- Pré-requisitos: `TASK-*` / nenhum

## Objetivo e resultado observável
[Definir o comportamento que esta task entrega.]

## Contexto mínimo e evidências
- Código/padrões existentes: [paths reais e razões]
- Interfaces relevantes: [contratos de request/evento/schema]
- Informações não verificadas: [o que precisa confirmar]

## Limites e isolamento
- Paths permitidos: [...] (reais/candidatos)
- Paths proibidos: [...]
- Riscos de edição compartilhada: [...]
- Contratos que precisam estar estáveis: [...]
- Fora de escopo: [...]

## Critérios de aceite da task
- [ ] **AC-001** (criterion): Dado [...], quando [...], então [...]
- [ ] **AC-003** (edge_case · permissions): Dado [...], quando [...], então [...]
- [ ] [teste de regressão pertinente]

## Casos de teste próprios (transcritos de `tests.md`)
Escreva **exatamente** estes casos, nos paths de teste permitidos, e faça-os passar:

| VER | Tipo · classe | Alvo | Entrada / setup | Resultado esperado | AC |
|---|---|---|---|---|---|
| VER-001 | unit · happy | `Service.metodo` | ... | ... | AC-001 |
| VER-002 | unit · error | `Service.metodo` | ... | erro `code=...` | AC-003 |

Comando existente (se conhecido): `...` / não verificado

## Regressões a manter verdes
| VER | Dona | Motivo |
|---|---|---|
| VER-0XX | TASK-0YY | [fronteira compartilhada] |

## Definição de entrega do implementador
- Mudança limitada ao escopo e revisável.
- Testes pertinentes executados quando viáveis; falhas e omissões relatadas.
- Resultado exclusivo por task em `tasks/<slug>/results/TASK-###/`.
- Não alterar `tasks.yaml` nem declarar a task `done`.
- Não realizar push/merge, migração destrutiva ou mudança de arquitetura sem autorização.

## Quando bloquear
[Informação de negócio ausente, dependência não integrada, restrição de segurança ou incompatibilidade real com o plano.]
