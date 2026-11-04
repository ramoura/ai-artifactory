# Dependências e execução paralela segura

## Distinções importantes

- **Dependência lógica:** uma task precisa do resultado de outra (ex.: consumidor usa um contrato ainda indefinido).
- **Conflito de edição:** tasks independentes podem modificar arquivos compartilhados (ex.: `pom.xml`, DTO comum, migrações, schema, fixture). Não crie dependência sem necessidade; registre risco de serialização e atribuição de ownership.
- **Dependência de integração:** a task precisa das alterações reais da predecessora já disponíveis no seu workspace; nunca confiar apenas no estado `implemented`.
- **Dependência externa:** acesso, aprovação, infraestrutura ou dados fora do repo; se não satisfeita, bloquear.

## Checklist de paralelismo

Para dois candidatos A e B, só executar ao mesmo tempo se TODOS:
1. `depends_on` de cada uma estão `done` com código integrado no workspace de execução.
2. Não há dependência causal A→B ou B→A.
3. Os contratos compartilhados foram estabilizados antes da edição.
4. Arquivos e recursos mutáveis não conflitam, ou existem worktrees isoladas **e estratégia de integração comprovada**.
5. O ambiente de testes não cria colisões em dados/portas/migrações.
6. Os limites corporativos permitem os processos e comandos necessários.

Worktrees isolam o diretório de trabalho, mas **não resolvem automaticamente conflitos de merge, mudanças de schema incompatíveis ou recursos de integração compartilhados**.

## Gates de revisão e integração

- `implemented` (resultado de worker) ≠ `in_review` ≠ `integrated` ≠ `done`.
- Registrar revisão independente com achados, evidências e severidade.
- Fixer só corrige os achados aceitos dentro do escopo; rerun de testes e nova revisão.
- Máximo de 3 rodadas de review/fix; conflito de instruções ou problema material persistente → `blocked` e decisão humana.
- Validar cenários que atravessam tasks, inclusive contratos, migrações e regressão.
- Nenhuma aprovação de PR/merge em branch protegida sem política corporativa e, quando exigido, aprovação humana.

## Exemplo apenas ilustrativo

```
TASK-001 (contrato)
  ├── TASK-002 (API)
  └── TASK-003 (cliente)

TASK-002 + TASK-003 done e integradas → /integration-validation (fora do DAG)
```

Se o contrato já existe e está estável, TASK-001 pode ser desnecessária. A `/integration-validation` não substitui reviewers independentes nem os testes dos implementadores.
