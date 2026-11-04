# Template do relatório final da história

> O conteúdo abaixo define a estrutura, não os resultados. Substituir placeholders apenas por evidência real.

# Validação de integração — [slug]

## Escopo e baseline
- História e plano: ...
- Modo: `validate | integrate-and-validate`
- Branch / commit / baseline: ... (`não verificado` quando indisponível)
- Integração realmente presente no workspace: sim / não / desconhecido
- Autorização de integração e restrições: ...

## Tasks e reviews
| Task | Revision | Code presente? | Review / rodada | Estado do gate |
| --- | --- | --- | --- | --- |
| TASK-... | ... | sim / não | approve / changes_requested / blocked | ... |

## Matriz de rastreabilidade
| Requisito | Aceite | Tasks | Verificações | Estado | Evidência |
| --- | --- | --- | --- | --- | --- |
| REQ-... | AC-... | TASK-... | VER-... | verified / failed / not_verified | ... |

## Verificações executadas no código integrado
(Para `VER-*` e2e de interface, cite as screenshots em `validation/screenshots/run-NN/` conferidas.)
| VER / checagem | Comando real (se houver) | Resultado | Evidência / motivo |
| --- | --- | --- | --- |
| VER-... | ... ou não executado | passed / failed / not_run | ... |

## Findings de integração
[Se aplicável, `VAL-###` com severity, evidência, REQ/AC/VER, TASK(s), causa provável, resolução esperada e next step]

## Riscos, pendências e bloqueios
[Ambientes ausentes, aprovações humanas, regressões preexistentes, requisitos sem verificação etc.]

## Decisão final e handoff
- Decisão: `accepted | changes_requested | blocked`
- Justificativa: ...
- Próximo responsável: orquestrador / task-review / task-breakdown / aprovação humana
- Próximo passo: ...
