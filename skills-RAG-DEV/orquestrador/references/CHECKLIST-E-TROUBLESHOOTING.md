# Checklist do orquestrador e troubleshooting

## Antes da execução

- [ ] Contratos Planning v2 válidos e `story.status: ready`, sem blockers.
- [ ] DAG válido e `TASK-*` com revision/scope/AC/VER reais.
- [ ] Riscos de concorrência, ownership e estratégia de integração declarados.
- [ ] Permissões e aprovação para modificar código confirmadas.
- [ ] Nenhuma task dependente liberada prematuramente.

## Para cada task

- [ ] `result.yaml`/`.md` existem e combinam com `task_id`/`revision`.
- [ ] Comandos e testes têm resultados reais, sem sucesso presumido.
- [ ] Diff real entregue ao revisor, com baseline e escopo identificáveis.
- [ ] Review independente com decisão e findings acionáveis.
- [ ] Correções re-revisadas; até 3 rodadas ou aprovação adicional.
- [ ] Mudanças integradas e eventual delta de conflito re-revisado.
- [ ] Somente o orquestrador registrou `done` em `tasks.yaml`.

## Final da história

- [ ] Workspace contém toda a implementação integrada e auditável.
- [ ] Todas as tasks relevantes `done` e sem review obsoleta.
- [ ] `integration-report.yaml` e `.md` atuais e rastreáveis.
- [ ] Todos os `REQ-*`, `AC-*`, `VER-*` cobertos e evidenciados.
- [ ] Achados `VAL-*` tratados e gate reexecutado após correções.
- [ ] Decisão técnica `accepted`, ou `changes_requested`/`blocked` com motivo.
- [ ] Aprovações humanas e restrições de PR/merge/deploy respeitadas.

## Troubleshooting

| Sintoma | Causa possível | Ação correta |
| --- | --- | --- |
| `/task-review` não é encontrada ou chama a skill nativa `code-review` | Instalação antiga com o nome `code-review` | A skill foi renomeada para `task-review` para não colidir com a nativa `/code-review`. Reinstale a pasta `task-review/` e remova a antiga |
| A sucessora falha mesmo com a predecessora `done` | O código ainda está em outra worktree/branch | Integrar a mudança no workspace da sucessora; nunca liberar só pelo status |
| Reviewer emite `blocked` por falta de diff | O resultado textual existe, mas o código não está acessível | Reunir o diff correto em workspace autorizado |
| Dois agentes tocaram o mesmo arquivo | Ownership/isolamento insuficiente | Suspender a concorrência, reconciliar e revisar o delta integrado |
| `result.yaml` diz `implemented`, mas os testes falham | A implementação não passou nos gates | Registrar a falha e delegar correção ou bloquear; não marcar `done` |
| `VAL-###` vem de incompatibilidade entre tasks | Contratos divergentes | Nova task ou replanejamento; não passar VAL direto para `--fix` |
| Agentes sem acesso a worktree | Política/ambiente restrito | Modo A, mantendo os mesmos artefatos |
| Agent Teams indisponível | Feature experimental ou política corporativa | Modo A, ou sessões separadas autorizadas |
| Terceira rodada de review continua reprovada | Problema persistente ou mal especificado | `blocked`, síntese das evidências e decisão humana/replanejamento |
| Esta skill foi carregada num subagente | Delegação indevida do papel | Parar; o orquestrador roda só no agente principal |
