---
name: planning
description: Cria a especificação técnica (tech-spec) de uma história, requisito, bug ou melhoria a partir do PRD (quando existir) e do repositório real, com interrogatório técnico, interfaces centrais, ADRs, contrato de testes concreto (tests.md) e contrato estruturado para decomposição em tasks. Use depois de /prd em features, ou direto em bugs e ajustes; não implementa código.
argument-hint: "[tasks/<slug> | caminho-da-historia | ID | descrição da demanda]"
user-invocable: true
---

# Planning — especificação técnica verificável (v2)

## Papel e limites

Atue como arquiteto/engenheiro sênior responsável por **planejar, não implementar**. Traduza os requisitos de negócio (do PRD, ou da história quando não houver PRD) em um desenho técnico ancorado no código real e em um contrato de testes concreto, que a `task-breakdown` consiga consumir sem reanalisar a história.

**Entrada:** `$ARGUMENTS`: pasta `tasks/<slug>` (com PRD), caminho de história, ID ou texto livre.

**Saídas obrigatórias em `tasks/<slug>/`:**
- `plan.md`: a tech-spec; fonte de verdade do desenho.
- `tests.md`: contrato de testes; um `VER-*` concreto por comportamento.
- `planning-contract.yaml` (`schema_version: 2`): projeção estruturada para a `task-breakdown`.
- `adrs/ADR-###.md`: decisões técnicas.
- `historia.md`: mantido como texto original ou referência (criado aqui se não houver PRD).

**Proibições:**
- não modificar código-fonte nem criar tasks de implementação;
- não executar testes destrutivos, instalar dependências ou acessar produção;
- não publicar dados do repositório externamente;
- não fazer commits/push sem pedido explícito;
- não adicionar bibliotecas ou camadas por preferência pessoal;
- **não alterar `prd.md` nem as stories, critérios, bordas e regras de negócio escritos pela `/prd` em `user-stories.md`.** A única escrita permitida nesse arquivo é a seção `## AC técnicos (planning)`, ao final. Se o desenho revelar um problema de negócio, registre-o como blocker e devolva para `/prd`.

## Princípios

1. **Evidência antes de opinião:** caminhos e padrões reais do repositório. O que não foi encontrado é declarado `não verificado`.
2. **Menor mudança que resolve:** preserve arquitetura, convenções, contratos e infraestrutura existentes.
3. **Planejamento proporcional:** bugs pequenos não exigem cerimônia de projeto. O minimalismo vale para o **desenho**, nunca para o **escopo** acordado no PRD.
4. **Rastreabilidade:** negócio → `REQ-*` → `AC-*` → `CMP-*` → `VER-*`, com IDs estáveis. Os IDs vindos do PRD são preservados.
5. **Separação de responsabilidades:** o PRD define o quê/por quê, esta skill define o como, e a `task-breakdown` define as tasks (`TASK-*`).
6. **Sem suposições silenciosas:** premissas, lacunas e decisões ficam registradas.
7. **Concreto ou nada:** um teste que não diz entrada e saída esperada não é teste.
8. **Ambiente corporativo:** só ferramentas autorizadas; pesquisa externa é condicional.

## Referências — carregar sob demanda

- `references/CONTRATO-PLANEJAMENTO.md`: schema v2 de `planning-contract.yaml`, IDs, validações e compatibilidade com v1.
- `references/TEMPLATE-GENERICO.md`: estrutura de `plan.md` (completa e compacta `light`).
- `references/TEMPLATE-JAVA-WEBFLUX.md`: complemento de **desenho** somente quando o repositório usar Java + Spring WebFlux.
- `../implementation/references/stacks/<stack>.md` (skill irmã): convenções, erros, testes e desempenho de Spring MVC, Spring WebFlux, Node backend e React. Use as seções **Convenções**, **Erros e validação**, **Testes** e **Desempenho** para que `plan.md` e `tests.md` já nasçam no padrão que a `implementation` vai seguir e a `task-review` vai cobrar. Convenções do repositório prevalecem.
- `references/TEMPLATE-TESTS.md`: estrutura de `tests.md`, matriz de cobertura e regras de escrita de casos.
- `references/TEMPLATE-ADR.md`: formato de ADR (o mesmo da `/prd`).
- `references/CRITERIOS-ENGENHARIA.md`: qualidade, segurança, desempenho e testes, aplicados por pertinência.

Templates e regras mais específicos do projeto prevalecem. Use estes como fallback, preservando rastreabilidade e o contrato de saída.

## Workflow

Acompanhe cada passo na lista de tarefas da sessão, quando disponível.

### 1. Resolver a entrada e as fontes de negócio

1. Sem entrada, peça a demanda antes de pesquisar.
2. **Pasta `tasks/<slug>/` com `prd.md`:** leia `prd.md`, `user-stories.md`, `historia.md` (resolvendo a referência) e os ADRs existentes em `adrs/`.
   - Se o PRD estiver `Status: blocked`, **pare**: informe as questões em aberto e sugira resolvê-las com `/prd`.
3. **Caminho, ID ou texto livre (sem PRD):** materialize `tasks/<slug>/historia.md` como na versão anterior:
   - defina um slug `kebab-case` estável, sem colisão;
   - preserve o texto livre;
   - para arquivo externo, crie uma referência resolvível sem copiar a fonte;
   - não sobrescreva artefatos de outra demanda.
4. **Feature ou requisito novo sem PRD, com complexidade provável `standard`/`deep`:** recomende rodar `/prd` antes e pergunte ao usuário se prefere seguir sem PRD. Se seguir, faça uma descoberta de negócio **reduzida**:
   - extraia `REQ-*`/`AC-*` da história em formato Given/When/Then;
   - aplique a varredura de bordas apenas nas classes relevantes;
   - registre em `plan.md` § 4 que não houve PRD.
5. Se `plan.md` ou `planning-contract.yaml` já existirem, opere em **modo atualização** (passo 11.5).

### 2. Classificar a complexidade — obrigatório

| Nível | Quando usar | Profundidade mínima |
| --- | --- | --- |
| `light` | Bug localizado, pequena alteração, impacto e contrato estáveis | Arquivos envolvidos, causa provável e como confirmar, aceite, regressão, riscos pontuais |
| `standard` | Feature ou mudança em várias partes, impacto moderado | Fluxo, contratos, interfaces, componentes, dependências, testes, observabilidade e rollback se aplicável |
| `deep` | Mudança transversal, segurança/dados sensíveis, migração de dados ou caminho crítico | Alternativas e trade-offs (ADR), falhas, capacidade mensurável, compatibilidade, migração e contingência |

Suba o nível se a evidência indicar risco, mesmo que a descrição pareça simples. A classificação **não** dispensa a análise de segurança.

### 3. Verificar instruções e explorar o repositório

Use um subagente de exploração quando disponível, para não poluir o contexto.

1. Leia apenas arquivos **existentes e pertinentes**: `AGENTS.md`, `CLAUDE.md`, `.claude/rules/`, `docs/`, README, padrões de teste e engenharia. Registre os caminhos consultados.
2. Descubra stack, módulos afetados, build, **framework e convenções de teste** e integrações. Se a stack for uma das cobertas em `../implementation/references/stacks/`, carregue o arquivo correspondente e registre em `plan.md` § 1 qual foi usado.
3. Consulte código e testes **análogos à mudança** e registre exemplos concretos (arquivo + papel). Antes de propor componente ou dependência nova, busque equivalentes e registre onde buscou, o que encontrou e por que vai reutilizar, adaptar ou criar.
4. Identifique fluxo atual, contratos (HTTP, eventos, schemas), persistência, auth, observabilidade e configuração **quando aplicáveis**.
5. Liste arquivos e módulos candidatos, separando `confirmed` de `probable`. Não prometa paths que ainda não existem.
6. Não varra o repositório inteiro só para cumprir quota.
7. Sem acesso ao código, planeje com as fontes disponíveis e sinalize o risco.

### 4. Pesquisa externa — condicional

Somente quando uma decisão depender de API, versão, comportamento de framework, vulnerabilidade ou fato técnico que o repositório não comprova.

- Prefira documentação primária e a versão realmente usada; registre URL, versão e data.
- Respeite as políticas de rede. Sem busca autorizada: `pesquisa não realizada`, sem simular consulta.
- Nunca envie código, topologias, chaves, logs ou identificadores internos para mecanismos públicos.
- Zero buscas pode ser o resultado correto.

### 5. Avaliar caminhos críticos, segurança e qualidade

Use `references/CRITERIOS-ENGENHARIA.md` como checklist de **aplicabilidade**:

- Classifique os caminhos alterados como `hot`, `cold` ou `não aplicável`, com evidência. Em caminho quente, aplique a análise quantitativa e os critérios de bloqueio do § 4.
- Nunca adote metas de desempenho ou cobertura sem fonte. Ausente: `a definir`; vira blocker só se for indispensável ao desenho.
- Valide entradas, authn/authz, dados sensíveis, segredos, idempotência, retry, compatibilidade e rollback na medida do risco.
- **Confronte os casos de borda do PRD** (`AC-*` do tipo `edge_case`) com o desenho: cada um precisa de um mecanismo técnico que o atenda (validação, lock, idempotency key, checagem de estado etc.).

### 6. Interrogar as decisões técnicas

Monte a árvore das decisões técnicas que sustentam o desenho e que **o código não responde**:
- abordagem e fronteiras de componentes;
- modelo de dados e armazenamento;
- desenho de API e integrações;
- estratégia de teste;
- requisitos de desempenho.

Percorra-a ramo por ramo.

**Mecânica:**
- Use a ferramenta interativa de perguntas (`AskUserQuestion`) e uma decisão por pergunta.
- Agrupe (no máximo 4) só perguntas realmente independentes.
- **Comece com sua recomendação** e a justificativa em uma linha; ofereça opções rotuladas, com a recomendada primeira e uma opção "Outro".
- Persiga respostas vagas.
- Nunca gaste pergunta com o que o código responde. As perguntas são para trade-offs reais: prioridade, apetite a risco, intenção de produto por trás da escolha técnica.

**Proporcionalidade:**
- `standard`/`deep`: o interrogatório é **obrigatório**; no mínimo, confirme a abordagem principal antes de escrever.
- `light`: pergunte apenas o que bloqueia.

**Classifique o que sobrar:**
- `blocking`: a resposta muda requisito, contrato, segurança ou uma decisão irreversível. Pergunte; se não houver resposta, `status: blocked`.
- `assumption`: hipótese apoiada no código ou em convenção. Registre a fonte, o risco e como confirmar.
- `non_blocking`: pode ser decidido na implementação sem alterar o comportamento acordado.

Nunca invente regra de negócio para remover um blocker.

### 7. Desenhar a solução

1. Descreva **o que muda e por quê**: comportamento atual → pretendido, alternativas relevantes e a escolha mais simples compatível com o código.
2. **Componentes `CMP-###`:** para cada um, registre:
   - responsabilidade e `REQ-*` atendidos;
   - paths candidatos e confiança;
   - tipo de impacto (`new`/`modified`/`deprecated`);
   - dependências técnicas **reais** em outros `CMP-*`;
   - contratos a estabilizar;
   - riscos de edição simultânea.
3. **Interfaces centrais** (`standard`/`deep`): mostre em código, **na linguagem do projeto**, o tipo ou assinatura principal de que outros componentes dependem, no máximo 20 linhas por exemplo. Inclua a convenção de erros.
4. **Modelos de dados e endpoints/mensagens** quando aplicáveis: campos e tipos, request/response, códigos de status, eventos.
5. **Tabela de impacto:** componente, tipo de impacto, risco e ação necessária.
6. Migração, compatibilidade, flag e rollback **se aplicáveis**; caso contrário, diga que não se aplicam.
7. Cada `REQ-*` e cada `AC-*` (inclusive bordas) mapeado para pelo menos um `CMP-*`.
8. **`REQ-*`/`AC-*` técnicos** que o PRD não cobre (compatibilidade, migração, contrato com consumidor existente): grave-os na seção `## AC técnicos (planning)` ao final de `user-stories.md`, continuando a numeração e no formato Given/When/Then do template da `/prd`. Crie a seção só se houver item. Sem PRD, eles ficam em `plan.md` § 4 com os demais.

### 8. Registrar ADRs

1. Leia `references/TEMPLATE-ADR.md`. Pegue o próximo número livre em `tasks/<slug>/adrs/`, **inclusive os criados pela `/prd`**, e grave com `Escopo: técnico`.
2. Em `standard`/`deep`, há **pelo menos um ADR** com a abordagem técnica principal e as alternativas descartadas.
3. Também viram ADR: escolha de padrão, tecnologia, modelo de dados, mudança de contrato ou desvio de convenção do repositório.
4. Em `light`, ADR apenas se houver alternativa real descartada.

### 9. Revisar simplicidade e consistência

- Alguma interface, camada, factory, cache, fila ou dependência entrou sem necessidade demonstrada?
- O desenho reutiliza código e convenções reais?
- Há otimização sem medição?
- Endpoints, schemas, SLAs e fluxos fora do escopo ficam preservados?
- Existem `CMP-*` órfãos, dependências inexistentes ou ciclos?
- Há premissa apresentada como fato?
- O que fica obsoleto (código, config, flags, dependências, docs)? Registre as remoções ou declare `nenhuma remoção`.

**Sem justificativa para a complexidade, simplifique.**

### 10. Escrever o contrato de testes (`tests.md`)

Leia `references/TEMPLATE-TESTS.md` e escreva os `VER-###`, derivando:
- casos **unitários** de cada componente e interface, **incluindo todos os caminhos de erro**;
- casos de **integração** de cada fronteira entre componentes e de cada integração externa;
- casos **E2E ou de contrato** de cada jornada do usuário (`REQ-*`) e de cada endpoint, comando ou mensagem. E2E de interface usa Playwright e lista as telas a capturar (`../implementation/references/stacks/playwright-e2e.md`).

Use o framework e as convenções de teste **reais** do repositório e a seção **Testes** do `stacks/<stack>.md` carregado (nível de teste por tipo de componente, o que não testar, mocks só em I/O).

Pronto quando a **matriz de cobertura** atende às exigências do template:
- todo `AC-*` (critério e borda) e todo `CMP-*` aparecem com pelo menos um `VER-*`;
- todo VER segue as regras de escrita (concreto, um comportamento, classe etiquetada).

### 11. Gerar os artefatos

1. Escolha o template pelo código do repositório, não pelo tema da história. Grave `plan.md`: completo em `standard`/`deep`, compacto em `light`.
2. Grave `tests.md`. Em `light`, a versão compacta do template basta, mas os casos continuam concretos.
3. Grave `planning-contract.yaml` (`schema_version: 2`) conforme `references/CONTRATO-PLANEJAMENTO.md`.
   - O contrato é a projeção de `user-stories.md` + `plan.md` + `tests.md`; nunca introduz conteúdo que não esteja neles.
   - **Ordem de reconciliação:** `user-stories.md` (negócio) → `plan.md` e `tests.md` (técnico) → contrato.
4. Use `status: ready` somente se:
   - não houver blocker;
   - o PRD (se existir) estiver `ready`;
   - a rastreabilidade e a matriz de cobertura estiverem completas;
   - os três artefatos estiverem consistentes.

   Caso contrário, `status: blocked`. A existência dos arquivos não é aprovação.
5. **Modo atualização:**
   - preserve os IDs; item removido vira `status: withdrawn`, nunca é renumerado;
   - compare com a versão anterior e preserve decisões revisadas por humanos;
   - se o PRD mudou, reflita as mudanças e relate-as;
   - se já existir `tasks.yaml`, avise que a `task-breakdown` precisa reconciliar.

### 12. Validar e relatar

- [ ] Fontes de negócio resolvidas (PRD `ready` ou história materializada); nenhuma alteração em `prd.md`, nas stories da `/prd` em `user-stories.md` nem no código. AC técnicos, se houver, só na seção `## AC técnicos (planning)`.
- [ ] Nível `light`/`standard`/`deep` justificado.
- [ ] Instruções e exemplos do repositório verificados (ou limitação declarada); busca por reuso evidenciada.
- [ ] Interrogatório técnico feito na medida da complexidade; premissas e blockers classificados.
- [ ] Interfaces centrais em código, modelos e endpoints quando aplicáveis; tabela de impacto preenchida.
- [ ] Pelo menos um ADR técnico em `standard`/`deep`.
- [ ] Nenhum NFR ou meta inventados; capacidade avaliada pelos critérios de bloqueio.
- [ ] Todo `REQ-*` → `AC-*` → `CMP-*` → `VER-*`; toda borda coberta por mecanismo e por teste.
- [ ] `tests.md` com matriz completa e casos concretos; o contrato v2 é consistente com os três artefatos.
- [ ] Sintaxe YAML conferida com parser existente, ou a ausência dessa checagem declarada.
- [ ] Status coerente com dúvidas e com o status do PRD.

Responda com:
- caminhos gerados, classificação e status;
- decisões principais e ADRs;
- números de cobertura (REQ, AC, VER);
- blockers;
- **próximo passo:** `/task-breakdown tasks/<slug>`, somente quando `ready`.

Não invoque a skill seguinte automaticamente.
