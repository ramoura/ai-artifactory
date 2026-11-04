# Matriz global de requisitos, aceite e verificações

## Fontes de verdade

1. `planning-contract.yaml`: catálogo de `REQ-*`, `AC-*` (critérios e bordas), `VER-*`; `tests.md` traz o caso concreto de cada `VER-*`.
2. `plan.md` e história: significado, riscos e comportamentos acordados.
3. `tasks.yaml` + specs: responsabilidade `TASK-*` e revisões.
4. Resultados/reviews: evidências locais, **não substituem** teste em código integrado.
5. Workspace final e testes nele executados: validação efetiva da entrega.

## Construção

Crie uma linha para **cada critério de aceite**. Não deduza que `REQ-001` está coberto apenas por uma task vinculada ao requisito.

| Requisito | Aceite | Tipo (criterion / edge · classe) | Tasks responsáveis | VER associadas | Status final | Evidência / lacuna |
| --- | --- | --- | --- | --- | --- | --- |
| REQ-... | AC-... | edge · permissions | TASK-... | VER-... | verified / failed / not_verified | ... |

Verifique cada `VER-*` (inclusive verificações que atendam mais de um `AC-*`). As verificações no contrato incluem um `type` e `expected_evidence`; respeite ambos.

## Cálculo das situações

- `verified`: comportamento confirmado no workspace integrado com evidência suficiente, diretamente ou por testes/inspeções compatíveis com o tipo de verificação.
- `failed`: verificação executada ou inspeção demonstra descumprimento de critério.
- `not_verified`: não executou, evidência insuficiente, ambiente indisponível ou implementação não observável. **Não é sucesso.**
- VER: `passed` somente para verificação realmente executada (por este validador ou evidência identificada como execução externa confiável); separar claramente autoria da execução; `failed` para falha observada; `not_run` com motivo.

## Cenários cruzados

- Fluxo completo cruzando API, camada de negócio e persistência/eventos.
- Erros e timeouts entre componentes; compatibilidade de contratos e serialização.
- Compatibilidade de migrações e versões; mecanismos de rollback quando exigidos.
- Autenticação, autorização, PII, idempotência, concorrência e observabilidade para fluxos afetados.
- Testes de regressão correspondentes aos critérios de aceite, não apenas testes novos isolados.

Não exigir E2E, carga, ferramentas extras ou cobertura percentual fixa onde não forem relevantes ou estiverem indisponíveis. Sinalize o risco de evidência faltante e aplique o gate correspondente.
