# Gates de integração segura e encaminhamento de falhas

## Dois modos, diferentes permissões

- **validate:** código em modo somente leitura, comandos de build/teste não destrutivos permitidos; relatórios em `validation/`. Se a integração ainda não existe, `blocked`.
- **integrate-and-validate:** requer autorização explícita, branch/workspace isolado, origem das tasks definida e estratégia de integração aprovada. **Não autoriza** merge em main, push, deploy, manipulação de produção, migrations destrutivas ou publicação.

## Antes de integrar

- Task precisa ter `revision` compatível, resultado do worker e review `approve` para o código apresentado; o coordenador define elegibilidade.
- Diferencie review do código de uma worktree vs. revisão do resultado pós-merge. Qualquer resolução que altere código exige nova revisão de seu diff.
- Commits/diffs têm origem e baseline verificáveis; não manipular branch de outro agente sem permissão.
- Confirme ownership de DTO/schema/migration/config compartilhada, conflitos semânticos e contratos.

## Durante e depois

- Preferir reunir mudanças em workspace de integração separado.
- Resolver somente conflitos mecânicos sem alterar semântica; conflitos de regra de negócio/contrato/arquitetura devem bloquear e voltar ao orquestrador.
- Executar verificações relevantes no **conjunto integrado**, não somente em branches individuais.
- Manter evidência real dos comandos e baseline. Limitações corporativas impedem ações não permitidas, não justificam inventar resultado.

## Falha após integração

```text
integration-validation: VAL-### (evidência + TASK-* possíveis)
     |
     +--> defeito isolável e dentro do escopo da TASK
     |      task-review <TASK> --context <integration-report.md>
     |      → round-NN.md com FIND-###
     |      → implementation <TASK> --fix <round-NN.md>
     |      → review independente → reintegração → validação global
     |
     +--> conflito cross-task, requisito/contrato novo, escopo insuficiente
            → orquestrador solicita replanejamento/task nova
            → revisão e execução autorizadas → reintegração → revalidação
```

**Não passe `VAL-*` diretamente para `implementation --fix`**: a versão existente espera um arquivo de review `FIND-*`.

## Proteções

- Um único escritor de `tasks.yaml` e outro único escritor dos relatórios globais; workers não disputam arquivos de coordenação.
- O DAG não contém task de validação final; esta skill é o gate. Exija que todas as tasks relevantes estejam `done` e integradas antes de aceitar a história.
- Se mudança de merge não revisada, review reprovado, task pendente, baseline ausente ou teste essencial impossível: não emitir `accepted`.
- Independência e aprovação humana permanecem necessárias em fluxos de alto risco conforme políticas.
