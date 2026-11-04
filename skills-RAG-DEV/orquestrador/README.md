# Orquestrador v2 (Claude Code)

## Instalação

```text
.claude/skills/orquestrador/
├── SKILL.md
└── references/
    ├── MAQUINA-ESTADOS-E-DAG.md
    ├── MODOS-PARALELISMO.md
    ├── REVIEW-FIX-E-VALIDACAO.md
    └── CHECKLIST-E-TROUBLESHOOTING.md
```

Copie a pasta `orquestrador/` para `.claude/skills/`, junto com `planning`, `task-breakdown`, `implementation`, `task-review` e `integration-validation`. As referências por stack (`implementation/references/stacks/`) são compartilhadas por `planning`, `implementation` e `task-review` via caminho relativo, então as três precisam estar instaladas lado a lado.

## Uso

Na sessão principal do Claude Code, depois de `/planning` e `/task-breakdown`:

```text
/orquestrador minha-historia
```

## Decisões de design

- `disable-model-invocation: true`: o modelo não carrega a skill sozinho, então nenhum subagente vira um segundo orquestrador por engano. Só o usuário a invoca.
- Sem `context: fork`: a skill precisa rodar no agente principal, que é quem consegue criar e coordenar subagentes.
- O `SKILL.md` traz só o laço de coordenação. Os detalhes ficam em `references/` e são lidos sob demanda.
- As restrições permanentes do projeto (sem push/deploy, sem evidência inventada) podem ficar também no `CLAUDE.md`, para valer fora da orquestração.
