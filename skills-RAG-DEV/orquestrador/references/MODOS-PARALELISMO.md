# Modos de paralelismo (sem nova instalação)

Escolha o modo no checkpoint de aprovação e declare-o ao usuário. Em dúvida, use o modo A.

## Modo A — sequencial (padrão mais previsível)

Uma sessão executa uma task de cada vez e aciona o review em contexto independente (subagente) quando disponível. É o ideal para testar os contratos, corrigir bugs localizados e trabalhar em projetos com muitos arquivos compartilhados. O orquestrador atualiza `tasks.yaml` ao final de cada gate.

## Modo B — múltiplas sessões/worktrees autorizadas

Use apenas se o ambiente permitir Git worktrees e várias sessões do Claude Code. Escolha até três tasks independentes e garanta o ownership dos arquivos. Exemplo **ilustrativo, apenas se autorizado**:

```bash
git worktree add ../repo-task-002 -b feature/task-002
git worktree add ../repo-task-003 -b feature/task-003
```

- Cada sessão recebe a spec correspondente; **só o orquestrador** escreve no registro canônico.
- Ao final, colete commits/diffs e artefatos de cada task usando a estratégia aprovada.
- Evite edição conjunta de `tasks.yaml`, `pom.xml`, migrations e outros caminhos compartilhados.
- Uma worktree **não** sincroniza automaticamente as mudanças de outra: arquivos de outra worktree não aparecem na sua. Colete-os explicitamente e não exponha relatórios entre worktrees por suposição.

## Modo C — Agent Teams (se disponível e permitido)

Recurso **experimental e desligado por padrão**. Ativação opcional, conforme a política local:

```bash
CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 claude
```

- No máximo 3 implementadores.
- Mantenha a correspondência entre as tasks internas da equipe e os IDs `TASK-*` de `tasks.yaml`. As tasks internas **não substituem** o registro da pipeline.
- Para trabalho fortemente sequencial ou com muito conflito, volte ao modo A.
- Não tente contornar bloqueios corporativos.

Documentação: https://code.claude.com/docs/en/agent-teams
