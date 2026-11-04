# Handoff Reviewer → Fixer — protocolo mínimo

O reviewer deve ser **independente do implementador**, quando possível em sessão/agente separado. Ele revisa o **diff real**, a spec, `plan.md`, contrato e resultados. Não basta perguntar ao implementador se o trabalho está correto.

## Review escrito (formato Markdown)

Caminho sugerido: `tasks/<slug>/reviews/TASK-###/round-NN.md`.

```markdown
# Review — TASK-003 — revision 1 — round 01

- Decisão: approve | changes_requested | blocked
- Escopo revisado: [diff, commits, arquivos, testes]
- Limitações: [o que não pôde ser verificado]

## Findings

### FIND-001 — [título]
- Severidade: critical | high | medium | low
- Status: open
- Evidência: [arquivo:linha, comportamento, teste, diferença com requisito]
- Requisito/critério: REQ-... / AC-... / VER-...
- Impacto: [erro e risco]
- Correção esperada: [resultado, sem prescrever refatoração excessiva]

## Verificações independentes
- [comandos e resultados reais ou justificativa de não execução]

## Conclusão
[aprovação, mudanças solicitadas ou bloqueio]
```

## Regras do reviewer

1. Vincular findings à evidência, não a gosto pessoal ou preferências arquiteturais.
2. Priorizar bugs funcionais, falhas de segurança, contratos quebrados, testes relevantes ausentes e regressão.
3. Não modificar código; a correção é responsabilidade do fixer.
4. Não aprovar sem comparar critérios de aceite e verificações reais; não afirmar testes executados se apenas constam no resultado do implementador.
5. Achado fora do escopo da task deve ser encaminhado como risco ou nova task, não forçado à correção oportunista.
6. Em mudanças sensíveis, revisão automatizada não substitui revisão humana exigida pela empresa.

## Regras do fixer

- Iniciar via `/implementation <TASK-###.md> --fix <review-round.md>` após liberação do coordenador.
- Responder a cada `FIND-*` como `fixed`/`unresolved`, com evidência real.
- Não alterar `planning-contract.yaml`, DAG ou outras tasks para satisfazer o review.
- Não alterar interface pública além do plano sem aprovação.
- Entregar atualização de `result.yaml` e `result.md`; o revisor recebe o novo diff e testa novamente.

## Política de ciclo

```
Implementação → review independente
  ├─ approve → gate de integração/testes → done (orquestrador)
  ├─ changes_requested → fixer → testes → review (próxima rodada)
  └─ blocked → decisão humana/replanejamento
```

No máximo **3 rodadas de review/fix** sem nova autorização. O loop pertence ao orquestrador. Se vários agentes usam worktrees, integrar/transportar a revisão e as mudanças antes de comparar resultados; não assumir que outra sessão vê os arquivos automaticamente.
