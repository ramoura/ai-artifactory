# Contrato da Task Review v2 — compatível com Implementation v2

## Fonte, formato e autoridade

**Input:** uma `TASK-###.md`, mesmo ID e `revision` em `tasks.yaml` e `results/TASK-###/result.yaml`, diff real disponível, `planning-contract.yaml` e plano.

**Output canônico:** `tasks/<slug>/reviews/TASK-###/round-NN.md`, para `NN = 01, 02, ...`. O documento Markdown é consumido diretamente por:

```text
/implementation tasks/<slug>/tasks/TASK-003.md --fix tasks/<slug>/reviews/TASK-003/round-01.md
```

A Task Review **não escreve `tasks.yaml`** nem `result.yaml`. Apenas o coordenador atualiza o status global. Não exigir `review.yaml` adicional.

## Estrutura obrigatória

```markdown
# Review — TASK-003 — revision 1 — round 01

- Decisão: changes_requested
- Task spec: tasks/exemplo/tasks/TASK-003.md
- Resultado analisado: tasks/exemplo/results/TASK-003/result.yaml
- Escopo revisado: [branch/commit/base reais, diff, arquivos]
- Revisão independente: [sessão/agente e sua separação, se possível]
- Limitações: [não executado, ausente, dúvidas]

## Cobertura de aceite e verificações
| ID | Situação | Evidência |
| --- | --- | --- |
| AC-001 | verified / partial / not_verified | ... |
| VER-001 | passed / failed / not_run | ... |

## Findings

### FIND-001 — Falha na validação de entrada
- Severidade: high
- Status: open
- Evidência: [arquivo/linha se verificada + comportamento reproduzível]
- Requisito/critério: REQ-001 / AC-001 / VER-001
- Impacto: [efeito real]
- Correção esperada: [resultado verificável]

## Verificações independentes
| Comando | Resultado | Evidência / motivo se não executado |
| --- | --- | --- |
| [comando real ou não executado] | passed / failed / not_run | ... |

## Histórico das rodadas (somente para round-02+)
- FIND-001: resolved / open — evidência

## Riscos observados fora de escopo
- [triagem separada, sem exigir correção oportunista]

## Conclusão e handoff
[aprovação ou alterações exigidas ou desbloqueio necessário]
```

**O exemplo é ilustrativo:** nunca copiar IDs, severidades ou decisões sem evidência. Se não houver achados, escreva `Nenhum finding material identificado.`. Caso não haja verificações executáveis, descreva `not_run` com motivo; não invente comando.

## Decisões

| Decisão | Uso | Encaminhamento |
| --- | --- | --- |
| `approve` | Sem findings materiais abertos e aceite essencial sustentado | Coordenador integra/valida antes de marcar `done` |
| `changes_requested` | Ao menos um finding aberto e corrigível no escopo | Coordenador chama Implementation v2 com `--fix` |
| `blocked` | Revisão inviável ou precisa de decisão/artefato essencial | Coordenador resolve bloqueio, não abre fix improvisado |

Findings com severidade `critical`, `high` ou `medium` abertos normalmente impedem `approve`; `low` pode ser melhoria não bloqueante quando a política permitir. Se a ausência de evidência for material, escolha `blocked` ou `changes_requested` conforme seja uma falha corrigível pelo implementador.

## Rastreabilidade e re-review

- Preserve `TASK-###`, `revision`, `round` e IDs `FIND-*` estáveis para o mesmo defeito.
- Não mude o significado de um finding entre rodadas; defeito novo recebe novo ID.
- A evidência de correção `review_findings` em `result.yaml` não é aceitação automática: valide o diff e as verificações de novo.
- Se uma revisão de task foi feita antes da integração, os testes e o review de integração posteriores ainda são exigidos.
- Revisor e implementador não devem ser o mesmo agente, se houver alternativa; ausência de independência deve ser informada.
- No máximo três rodadas sem nova autorização do coordenador/usuário.
