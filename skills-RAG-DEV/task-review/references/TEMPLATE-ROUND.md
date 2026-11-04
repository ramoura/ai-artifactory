# Template de rodada de review

> Substituir placeholders a partir de evidências reais. Não copiar os exemplos de finding do contrato como se fossem descobertas.

# Review — TASK-### — revision [N] — round [NN]

- Decisão: `approve | changes_requested | blocked`
- Task spec: `tasks/<slug>/tasks/TASK-###.md`
- Resultado analisado: `tasks/<slug>/results/TASK-###/result.yaml`
- Escopo revisado: [base confiável, diff, caminhos e commit/branch quando reais]
- Revisão independente: [agente/sessão ou limitação]
- Limitações: [o que não foi validado]

## Cobertura de aceite e verificações
| ID | Situação | Evidência |
| --- | --- | --- |
| AC-... | verified / partial / not_verified | ... |
| VER-... | passed / failed / not_run | ... |

## Findings
[Se houver, usar `### FIND-### — título` com Severidade, Status, Evidência, Requisito/critério, Impacto, Correção esperada. Caso contrário: Nenhum finding material identificado.]

## Verificações independentes
| Comando | Resultado | Evidência / motivo |
| --- | --- | --- |
| ... | passed / failed / not_run | ... |

## Histórico das rodadas
[Para re-review: finding anterior resolved/open, evidência e novos achados]

## Riscos observados fora de escopo
[Triagem, não bloquear indevidamente]

## Conclusão e handoff
[Decisão fundamentada e próximo responsável]
