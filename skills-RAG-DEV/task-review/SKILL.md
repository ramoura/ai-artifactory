---
name: task-review
description: Revisa independentemente a implementação ou correção de exatamente uma TASK-### da pipeline Planning v2, compara o diff real com requisitos, testes e contratos, e produz findings acionáveis compatíveis com implementation --fix. Não altera código.
argument-hint: "[tasks/<slug>/tasks/TASK-###.md] [--round NN] [--context caminho-de-relatorio]"
user-invocable: true
---

# Task Review — revisão independente por task (v2)

## Responsabilidade

Atue como **revisor independente** do agente que implementou a task. Sua função é identificar defeitos comprováveis e verificar o alinhamento entre implementação, contrato e evidências. **Não escreva código nem corrija findings**; o agente `implementation --fix` é responsável pelas correções.

**Entrada:** `$ARGUMENTS` com uma única `tasks/<slug>/tasks/TASK-###.md`; opções:
- `--round NN`: número da rodada de review (opcional; se ausente, determinar a próxima rodada livre a partir dos arquivos visíveis e da autorização do coordenador).
- `--context <arquivo>`: evidência adicional, como `tasks/<slug>/validation/integration-report.md` (opcional). Não transforma esta review em validação da história.

**Saída canônica:** `tasks/<slug>/reviews/TASK-###/round-NN.md` usando `references/CONTRATO-REVIEW.md`. Este é o arquivo que a Implementation v2 consome como argumento `--fix`.

**Escrita exclusiva:** apenas o arquivo da rodada atribuída. Não alterar `tasks.yaml`, `plan.md`, `planning-contract.yaml`, `TASK-###.md`, `results/`, relatórios de outras tasks, código-fonte, Git ou decisões anteriores. Não criar commit, branch, PR, push, merge, deploy ou instalar ferramentas.

**Autoridade:** emita `approve`, `changes_requested` ou `blocked` para esta task/rodada. **Só o orquestrador** atualiza o status canônico em `tasks.yaml` e decide se a task está `done` após integração e gates.

## Referências — ler quando necessário

- `references/CONTRATO-REVIEW.md` — formato do Markdown e semântica de decisão (`FIND-*`). **Obrigatória.**
- `references/CHECKLIST-REVIEW.md` — verificação proporcional por categoria e stack.
- `references/TEMPLATE-ROUND.md` — exemplo de estrutura para a saída; jamais copiar achados fictícios.
- Na skill irmã: `../implementation/references/PROTOCOLO-REVIEW-FIX.md`, se disponível; o contrato local foi desenhado para ser compatível.
- Na skill irmã: `../implementation/references/stacks/<stack>.md`, quando a task tocar Spring MVC, Spring WebFlux, Node backend ou React. Use a seção **Checagens para review** e as **proibições**; os **defaults** não geram finding em código que segue uma convenção já presente no repositório.

## Workflow

### 1. Resolver o alvo e checar pré-condições

1. Identifique **uma única** spec `TASK-###.md` no caminho informado; não assuma que a história toda está sendo revisada.
2. Leia `tasks/<slug>/tasks.yaml` **somente leitura** e confirme `task_id`, `revision`, escopo, dependências e estágio autorizado pelo orquestrador (normalmente `in_review`). Estado desatualizado em worktree exige confirmação; nunca o atualize.
3. Leia `tasks/<slug>/results/TASK-###/result.yaml` e `result.md`, a spec, `planning-contract.yaml`, `tests.md` e partes pertinentes de `plan.md` (interfaces e ADRs citados). Use `user-stories.md`/`prd.md` (regras `RN-*`) e `historia.md`, quando presentes, para confirmar a intenção de negócio.
4. Verifique que o resultado se refere à **mesma revisão** e que o worker reportou `status: implemented` e `review_ready: true`. Se não, explique por que não é revisável e emita `blocked`.
5. Identifique a **rodada correta**. Não sobrescreva rodada anterior; na dúvida sobre número ou limite, interrompa e peça ao coordenador. A política vigente limita a três rodadas de review/fix, salvo autorização adicional.
6. Se houver contexto externo (`--context`), confirme que pertence à mesma história e vincule seus achados às tasks afetadas sem reescrever outros relatórios.

### 2. Definir a base real da revisão

1. Inspecione o código atual e **diff/commits reais** da task. Obtenha base confiável (branch, commit, diff disponibilizado pelo coordenador ou comparação comprovada). **Não deduza o diff de uma descrição em `result.md`.**
2. Em worktrees, confirme que o workspace contém a implementação/correção esperada. Se o código não chegou ao workspace do reviewer, emita `blocked`; não marque `approve` com base apenas no relato.
3. Leia os arquivos realmente tocados e seus pontos de chamada, testes, configurações e contratos relevantes. Respeite as permissões corporativas.
4. Diferencie mudanças da task de modificações preexistentes ou de outros agentes. Se não for possível atribuir corretamente uma mudança material, registre a limitação e solicite reconciliação.
5. Verifique se há mudanças fora de `scope.allowed_paths`, em `scope.avoid_paths` ou em arquivos compartilhados não autorizados; informe ao coordenador.

### 3. Revisar comportamento e regressões

Avalie proporcionalmente:
- **Correção de negócio:** cada `REQ-*` / `AC-*` da task, **inclusive os `edge_case`** (permissão, concorrência, repetição, estado etc.), e as regras `RN-*` aplicáveis.
- **Contratos e compatibilidade:** APIs, eventos, schemas, serialização, migrações, consumidores e versionamento.
- **Segurança:** autenticação/autorização, validação, segredos, PII, injeção, logs e controles necessários.
- **Concorrência e resiliência:** idempotência, retries, timeout, tratamento de exceções, recursos, reatividade/backpressure quando pertinentes.
- **Testes e observabilidade:** cada `VER-*` próprio da task existe na suíte, com o alvo, a entrada e a saída de `tests.md` (caso ausente ou enfraquecido vira finding); testes de regressão, qualidade do assert, efeitos colaterais, métricas/logging exigidos pelo plano.
- **Manutenibilidade:** simplicidade, integração com convenções do projeto, ausência de abstrações e dependências desnecessárias.

Use `references/CHECKLIST-REVIEW.md` e, para a stack identificada, a seção **Checagens para review** de `../implementation/references/stacks/<stack>.md`. Uma **proibição** da stack violada em código novo é finding (`medium` ou acima conforme impacto); um **default** não seguido só é finding se o repositório não tiver convenção própria e o desvio tiver efeito demonstrável. Não imponha metas inventadas de RPS, cobertura, estilo ou performance.

### 4. Verificar evidências independentemente

1. Para cada `AC-*` e `VER-*`, confronte a implementação com a evidência em `result.yaml` / `result.md`.
2. Se o ambiente autorizado permitir, execute verificações **não destrutivas e relevantes** (testes, lint, inspeção, build) sem instalar ferramentas, alterar produção ou mudar arquivos.
3. Registre cada comando efetivamente executado e o resultado observável. Comando descrito pelo implementador **não foi executado pelo reviewer**, salvo execução própria comprovada.
   - **Screenshots de `VER-*` e2e:** abra cada imagem listada em `verification[].artifacts` (a ferramenta de leitura exibe PNG) e confirme que a tela mostra o estado descrito em `tests.md`. Imagem ausente ou que não corresponde ao estado torna o `VER-*` `not_verified` e, se o comportamento for essencial, vira finding. Registre na tabela de cobertura quais imagens foram conferidas.
4. Quando um teste essencial não estiver disponível, registre `not_run` e motivo. Ausência de evidência essencial impede `approve`; verificações não essenciais podem ficar como limitações explícitas conforme risco.
5. Não afirmar `passed`, vulnerabilidade, linha ou comportamento sem prova. Se necessário, faça um finding por ausência de validação, com referência objetiva.

### 5. Produzir findings acionáveis

Um finding precisa conter:
- ID estável **por task** `FIND-001`, `FIND-002`, ...; se persistir em nova rodada, **reuse o mesmo ID** em vez de inventar outro.
- Título, `critical | high | medium | low`, `status: open` (ou `resolved` para rechecagens documentadas).
- Local e evidência reproduzível (path/linha **se verificados**; diff, condição e consequência).
- `REQ-*`, `AC-*`, `VER-*` relacionados, **quando existentes**.
- Impacto e **correção esperada como comportamento**, sem prescrever refatoração extensa.
- Distinção entre defeito introduzido pela task e risco preexistente/fora de escopo.

Não produza findings cosméticos sem impacto demonstrável nem reporte preferência pessoal como bug. Findings preexistentes fora de escopo devem ir em "Riscos observados" para triagem posterior.

### 6. Decidir com critérios transparentes

- `approve`: implementação revisável, requisitos e verificações essenciais sustentados por evidência, sem findings materiais abertos. `low` não bloqueante pode ser registrado como recomendação se a política permitir.
- `changes_requested`: existem findings `open` acionáveis no escopo (bugs, incompatibilidade, risco relevante, teste essencial corrigível), com evidência suficiente para `implementation --fix`.
- `blocked`: sem diff confiável, revisão/referências inconsistentes, falta de acesso crítico, dependência de decisão de negócio/arquitetura ou impossibilidade material de concluir revisão. Informe precisamente o desbloqueio necessário.

Não confundir `approve` do reviewer com `done` no DAG nem com aceite final da história. Em mudanças sensíveis, destaque a necessidade de aprovação humana prevista pelo projeto.

### 7. Gravar a rodada sem perder histórico

1. Leia `references/CONTRATO-REVIEW.md` e produza `reviews/TASK-###/round-NN.md` com decisão, task/revisão/rodada, base do diff, verificações, findings e próximo passo.
2. Em `round-02+`, compare findings anteriores: liste resolvidos, persistentes e novos, preservando IDs. Não sobrescreva `round-01.md`.
3. Se as evidências de correção foram produzidas pelo Fixer, confirme no código; não aceitar `review_findings: fixed` sem validação independente.
4. Não edite nenhum arquivo compartilhado ou de código. Se não houver permissão para gravar o relatório, devolva o conteúdo ao orquestrador e informe que o arquivo **não foi salvo**.
5. Reporte: decisão, findings por severidade, verificações feitas/não feitas, limites e caminho do relatório.

## Checklist de saída

- [ ] Uma única task; ID/revision/result/rodada compatíveis.
- [ ] Workspace/diff real identificados; mudança atribuível à task.
- [ ] `AC-*` e `VER-*` relevantes revisados com evidência ou lacuna explícita.
- [ ] Findings `FIND-*` objetivos e acionáveis, com IDs estáveis entre rodadas.
- [ ] Decisão compatível com findings, blockers e evidências.
- [ ] Arquivo canônico `round-NN.md` salvo sem sobrescrever anteriores.
- [ ] Nenhum arquivo de código, controle de DAG ou resultado do implementador alterado.

**Handoff:** `changes_requested` → coordenador libera `/implementation <spec> --fix <reviews/TASK-###/round-NN.md>`; `approve` → coordenador avalia gates de integração; `blocked` → coordenador resolve causa ou pede decisão humana.
