---
name: integration-validation
description: Valida a história inteira após integrar implementações revisadas, verificando dependências, compatibilidade, testes e rastreabilidade REQ/AC/VER; produz decisão final e findings de integração para o orquestrador. Não aprova merge/deploy.
argument-hint: "[tasks/<slug>] [--mode validate|integrate-and-validate]"
user-invocable: true
---

# Integration Validation — gate final da história (v2)

## Responsabilidade e limites

Atue como **validador independente da entrega integrada**, diferente do reviewer por task. Seu objeto é **a história completa**, não uma única `TASK-###`. Verifique se o conjunto das mudanças implementa todos os critérios de aceite e se componentes/agentes convivem corretamente.

**Entrada:** `$ARGUMENTS` com `tasks/<slug>/` ou o caminho para o `planning-contract.yaml` da história, seguido opcionalmente de `--mode`:

- `validate` (**padrão**): **somente leitura do código**, com execução de verificações não destrutivas e gravação de relatórios na pasta `validation/`.
- `integrate-and-validate`: somente quando o orquestrador **autorizar explicitamente**, operar em workspace/branch de integração isolado e previamente definido, usando a estratégia de coleta/integração aprovada. Não implica permissão para push, merge na branch principal, deploy ou resolução arbitrária de conflitos de negócio.

**Leituras obrigatórias:**
- `tasks/<slug>/historia.md`, `prd.md` e `user-stories.md`, se existirem; `plan.md`; `tests.md` (v2); `planning-contract.yaml` (`schema_version: 2` ou `1`).
- `tasks/<slug>/tasks.yaml` e `execution-plan.md`.
- Specs `tasks/TASK-###.md`, resultados `results/TASK-###/result.yaml` + `.md`, reviews `reviews/TASK-###/round-NN.md` relevantes.
- Código e testes **efetivamente disponíveis no workspace de integração**, além de instruções pertinentes do repositório (`AGENTS.md`, `CLAUDE.md` etc.).

**Saídas exclusivas:**
- `tasks/<slug>/validation/integration-report.yaml` — decisão e evidências estruturadas, conforme `references/CONTRATO-VALIDACAO.md`.
- `tasks/<slug>/validation/integration-report.md` — relatório legível e matriz completa `REQ-* → AC-* → VER-* → TASK-*`.

**Não alterar:** `tasks.yaml`, especificações `TASK-*`, planejamento, reviews, resultados de implementadores ou artefatos de outras histórias. Não marcar tasks `done`, não publicar PR, não fazer merge para a branch principal, push, deploy, migração destrutiva ou instalação sem autorização. **A decisão final desta skill não substitui aprovação humana exigida pela empresa.**

## Referências

- `references/CONTRATO-VALIDACAO.md` — schema obrigatório de relatório e política de decisões.
- `references/MATRIZ-EVIDENCIAS.md` — cálculo da cobertura e avaliação de verificações.
- `references/GATES-INTEGRACAO.md` — isolamento de branches, conflitos, execução, segurança e handoff de falhas.
- `references/TEMPLATE-RELATORIO.md` — estrutura do Markdown e exemplos não normativos.

## Workflow

### 1. Resolver a história e o modo de operação

1. Extraia o `slug` e confira que os artefatos são da **mesma história**. Não invente arquivos ou integre histórias não relacionadas.
2. Confira `story.status: ready` e `blockers: []` no `planning-contract.yaml`; caso contrário, `blocked`.
3. Confirme que o DAG em `tasks.yaml` tem IDs e revisões atuais, dependências válidas e sem ciclos. Esta skill é o **único** gate final da história; o DAG não contém task de validação. Se encontrar uma task `kind: validation` ou `kind: integration` (registro legado), trate-a como task comum e não a use como substituto desta validação.
4. Defina `validate` ou `integrate-and-validate`; anote autorização real, branch/workspace de destino e limitações. Sem autorização explícita para integração, preserve o modo `validate`.
5. Verifique se as tasks relevantes estão `done`, ou se o orquestrador forneceu autorização e evidência de que todo o trabalho está revisado/integrado para o gate final. Se houver task pendente, review rejeitado ou revisão obsoleta, **não aceite** a história.

### 2. Verificar a integração real

1. Confirme Git/workspace atual, origem da baseline, commits ou diffs das tasks e integração presente. **Não assumir que branches/worktrees compartilham mudanças automaticamente.**
2. Compare o código presente com os relatórios de execução e rounds de review **aprovados**. A aprovação de review anterior a um merge não prova que o conjunto final é compatível.
3. No modo `integrate-and-validate`, somente com autorização e estratégia explícitas:
   - reúna alterações de worktrees/branches no **workspace de integração isolado**;
   - confira conflitos e ownership de arquivos, migrações, schemas e contratos;
   - resolva apenas conflitos mecânicos sem ambiguidade e dentro da autorização;
   - em conflitos semânticos ou mudanças de requisito, **bloqueie** e devolva ao orquestrador;
   - qualquer código alterado para resolver conflito **precisa voltar à revisão independente** antes de ser considerado aprovado.
4. Se a integração ainda não ocorreu no modo `validate`, reporte `blocked` por ausência de baseline integrada; não compare worktrees isoladas como se fossem o sistema final.
5. Verifique que a integração não introduziu arquivos fora de escopo, alterações imprevistas de infraestrutura, segredos ou efeitos colaterais.

### 3. Construir matriz de rastreabilidade global

1. Liste **todos** os requisitos `REQ-*` e critérios `AC-*` ativos do `planning-contract.yaml`, **critérios e casos de borda** (indique `kind`/`edge_class`), com as verificações `VER-*` relacionadas, a task dona de cada VER e as tasks que os implementaram. Ignore itens `withdrawn`.
2. Para cada `AC-*`, confira evidência de comportamento no **código integrado**, não apenas resultado unitário de cada worker.
3. Para cada `VER-*`, identifique teste/checagem, comando autorizado e evidência de execução ou motivo de `not_run`.
4. Inclua testes cruzando componentes, compatibilidade de contratos e regressões pertinentes, conforme plano; não invente métricas, mocks ou ambientes indisponíveis.
5. Um `AC-*` sem cobertura ou evidência suficiente deve ser `not_verified` ou `failed`, nunca `verified` por inferência.
6. Use `references/MATRIZ-EVIDENCIAS.md` para consistência e níveis de cobertura. Não confundir "task feita" com "requisito validado".

### 4. Executar gates de qualidade autorizados

Execução progressiva, proporcional a risco e ambiente:

- Verificações de diff/escopo/configuração e análise das alterações de merge.
- Build, lint/static checks e testes unitários pertinentes.
- Testes de integração/contrato, regressão e E2E quando aplicáveis e disponíveis. E2E de interface com Playwright no código integrado, salvando as capturas em `tasks/<slug>/validation/screenshots/run-NN/` (mesmo `NN` do snapshot do relatório) e listando-as em `verification[].artifacts`. As screenshots dos workers em `results/` são evidência histórica; a aceitação exige captura sobre o conjunto integrado ou `not_run` justificado.
- Cenários de erro, limites, compatibilidade, resiliência e segurança que o plano exigir.
- Para fluxos críticos, desempenho/carga apenas se houver ambiente e autorização adequados; não produzir carga contra sistemas corporativos reais por padrão.

Registre **comandos realmente executados**, status `passed | failed | not_run` e resultado observável. Quando testes dependem de infraestrutura ausente, registre limitações e determine se impedem aceite; jamais simule sucesso. Não instalar ferramenta, abrir rede, criar infra ou acessar produção sem autorização.

### 5. Tratar achados de integração e falhas

Quando encontrar problemas:

1. Crie `VAL-001`, `VAL-002` etc. **estáveis entre reexecuções**, com severidade `critical | high | medium | low`, evidência, `REQ/AC/VER` envolvidos, tasks candidatas e correção esperada.
2. Diferencie falha introduzida por uma task, incompatibilidade entre tasks, falha preexistente, problema de ambiente e requisito ausente.
3. **Não edite tasks nem implemente correções** no modo `validate`. Informe ao orquestrador a responsabilidade provável e a evidência, sem atribuir certeza artificial quando houver vários responsáveis.
4. Para corrigir via `implementation --fix`, **não passe `integration-report.md` diretamente como review**. O orquestrador encaminha o achado à `/task-review` de uma task afetada (com `--context tasks/<slug>/validation/integration-report.md`) para gerar `round-NN.md` e `FIND-*` compatíveis; ou solicita uma nova task via task-breakdown se a falha cruzar escopos.
5. Se a correção afetar contrato, arquitetura ou requisitos, pare para replanejamento/decisão; não ampliar `scope.allowed_paths` por conveniência.
6. Após qualquer correção, o coordenador precisa reintegrar as alterações, rever código afetado e solicitar **nova validação global**. Não tratar relatório antigo como válido.

### 6. Emitir decisão final

- `accepted`: todas as tasks relevantes estão integradas e revisadas, `AC-*` essenciais verificados, verificações obrigatórias suficientes, sem falhas materiais abertas, sem bloqueio corporativo conhecido.
- `changes_requested`: há defeitos comprovados e corrigíveis (por task ou entre tasks), com `VAL-*` e encaminhamento claro.
- `blocked`: artefatos, permissões, integração, aprovação obrigatória, ambiente/evidência essencial ou decisão de negócio indisponíveis. Indique precisamente a condição de desbloqueio.

`accepted` significa **gate técnico da história atendido**, não aprovação automática para merge, deploy ou publicação. Se faltar aprovação humana prevista, registre a pendência e não represente a entrega como aprovada para publicação.

### 7. Persistir relatório e entregar ao orquestrador

1. Preencha `integration-report.yaml` conforme `references/CONTRATO-VALIDACAO.md`; `null` para campos não verificados, listas vazias quando apropriado, sem fabricar hashes, comandos ou resultados.
2. Gere `integration-report.md` com: baseline integrada, lista de tasks/revisões, reviews aprovados, matriz de cobertura `REQ→AC→VER→TASK`, comandos/resultados, `VAL-*`, riscos, blockers e recomendação final.
3. Na reexecução, preserve histórico **sem pedir autorização a cada rodada**: se já existir relatório canônico, antes de substituí-lo copie ambos os arquivos anteriores (se presentes) para o próximo par livre `integration-report-run-NN.yaml/.md`, dentro de `validation/`; nunca sobrescreva snapshots. Em seguida grave os relatórios canônicos atualizados. Se houver escritor concorrente, erro na cópia ou conflito de versão, **bloqueie** a escrita e peça reconciliação ao orquestrador. O relatório canônico reflete a avaliação mais recente, inclusive se reprovada.
4. Escreva exclusivamente em `validation/` (relatórios e `screenshots/run-NN/`), usando coordenação de único escritor para evitar concorrência.
5. Responda com decisão, tasks/revisões examinadas, cobertura comprovada, testes executados/não executados, findings, blockers e caminho dos relatórios.

## Checklist final

- [ ] Mesmo `slug`, `schema_version` do contrato (2 ou 1) reconhecido, IDs reais e revisões coerentes.
- [ ] Workspace representa a **integração real** das tasks, ou decisão `blocked`.
- [ ] Reviews por task consultados e de rodadas compatíveis.
- [ ] Matriz contém todo `REQ-*`, `AC-*`, `VER-*` aplicável e tasks relacionadas.
- [ ] Testes reportados com comando/resultado reais ou `not_run` justificado.
- [ ] Findings `VAL-*` úteis; encaminhamento correto para reviewer/fixer ou replanejamento.
- [ ] Sem código não revisado declarado pronto; sem falso aceite por conclusão de tasks.
- [ ] Relatórios gravados sem alterar `tasks.yaml`, plano, reviews ou resultados dos workers.
- [ ] Nenhum push, merge na branch principal, deploy ou ação destrutiva sem autorização.

**Handoff:** `accepted` → coordenador avalia PR/approval gates; `changes_requested` → coordenador encaminha `VAL-*` a task-review ou task-breakdown; `blocked` → coordenador resolve a condição pendente e repete o gate.
