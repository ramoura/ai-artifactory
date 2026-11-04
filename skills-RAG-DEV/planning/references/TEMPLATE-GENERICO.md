# Template de `tasks/<slug>/plan.md` — genérico (tech-spec)

> O `plan.md` é a tech-spec: define o **como**. O **o quê/por quê** vive em `prd.md`/`user-stories.md`; referencie-os por ID e seção, sem duplicar o contexto de negócio. Substitua os placeholders. Em `standard`/`deep`, use todas as seções abaixo e `Não aplicável — [motivo]` quando necessário. Para `light`, use o formato compacto ao final; regras específicas do projeto prevalecem.

# Plano de implementação — [título]

## 1. Identificação e estado
- Demanda/origem: [caminho ou descrição]
- PRD: `prd.md` + `user-stories.md` [ready] — ou `sem PRD` com o motivo
- Slug: [slug]
- Tipo: [feature | bug | melhoria | requisito]
- Complexidade: [light | standard | deep] — [justificativa]
- Status: [ready | blocked] — [motivo]
- Stack e módulos: [com base no repositório]

## 2. Problema, objetivo e escopo
- Situação atual / reprodução (bugs): ...
- Resultado pretendido: ...
- No escopo: ...
- Fora do escopo e comportamentos preservados: ...
- Restrições relevantes: ...

## 3. Evidências e padrões existentes
| Evidência | Caminho ou origem | Impacto no desenho |
|---|---|---|
| ... | ... | ... |

Para componentes/dependências novos, inclua busca por reuso, resultado e justificativa da escolha.

## 4. Requisitos e critérios de aceite
**Com PRD:** não repita as stories. Indexe `user-stories.md`, inclusive os `REQ-*`/`AC-*` **técnicos** que esta planning acrescentou na seção `## AC técnicos (planning)` daquele arquivo (ex.: compatibilidade, migração). O texto Given/When/Then vive lá; aqui só a tabela de índice.

| Requisito | Origem | Critérios (criterion) | Bordas (edge_case · classe) |
|---|---|---|---|
| REQ-001 | user-stories.md | AC-001, AC-002 | AC-003 (permissions), AC-004 (repetition) |

**Sem PRD:** descreva cada `REQ-*` e cada `AC-###` em Given/When/Then, aplicando as classes de borda relevantes (`invalid_input`, `empty`, `limits`, `permissions`, `concurrency`, `interruption`, `repetition`, `ordering`, `state`, `scale`).

## 5. Solução técnica e componentes
- Fluxo atual → proposto (texto ou diagrama simples): ...
- Decisão recomendada e por quê: ...
- Alternativas relevantes descartadas (em `deep`, incluir trade-offs): ...

| Componente | Responsabilidade | Requisitos | Arquivos/módulos candidatos | Depende de | Potencial conflito |
|---|---|---|---|---|---|
| CMP-001 | ... | REQ-001 | ... | — | ... |

Não gere IDs de task aqui. Os componentes são unidades **lógicas**, não garantias de paralelismo.

### 5.1 Interfaces centrais
Tipo/assinatura principal de que outros componentes dependem, **na linguagem do projeto**, no máximo 20 linhas por exemplo, com a convenção de erros:

```[linguagem]
// CMP-001 — [nome]
```

### 5.2 Modelos de dados
Entidades e campos com tipos, relacionamentos, request/response, schema ou estrutura de armazenamento — **quando aplicável**.

### 5.3 Endpoints, comandos e mensagens
| Método/Tipo | Path/Tópico | Request (campos obrigatórios) | Respostas e códigos | AC |
|---|---|---|---|---|
| POST | /... | ... | 201 ...; 400 code=...; 403 ... | AC-001, AC-003 |

### 5.4 Mecanismo para cada caso de borda
| AC (edge_case) | Classe | Mecanismo técnico | CMP |
|---|---|---|---|
| AC-003 | permissions | [checagem de papel em ...] | CMP-002 |

### 5.5 Análise de impacto
| Componente/área | Impacto | Descrição e risco | Ação necessária |
|---|---|---|---|
| ... | new / modified / deprecated | ... | ... |

## 6. Contratos e integração
- API/eventos/schemas: ...
- Persistência e migração: ...
- Compatibilidade/consumidores: ...
- Dependências externas: ...

## 7. Requisitos não funcionais e operação
- Segurança/dados sensíveis: ...
- Caminhos quentes/frios (com fonte): ...
- Metas conhecidas, fontes e lacunas (com efeito sobre `ready`): ...
- Se alterar capacidade: demanda atual/projetada por dependente ou recurso, limites, premissas e comportamento sob saturação: ...
- Observabilidade, resiliência, rollout e rollback: ...

## 8. Estratégia de validação
Somente a **estratégia**; todo caso concreto vive em `tests.md`:
- Frameworks, harness e fixtures reais do repositório; fakes apenas em fronteiras de I/O.
- O que cada nível (unit / integração / contrato / e2e) cobre nesta feature e como roda.
- Dependências de ambiente/dados das suítes de integração e e2e.
- Resumo da cobertura: N `AC-*` → M `VER-*`; link para a matriz em `tests.md`.

## 8.1 Decisões (ADRs)
- [ADR-###: título](adrs/ADR-###.md) — escopo (produto/técnico) — resumo em uma linha

## 9. Dependências, riscos, premissas e perguntas
- Dependências reais entre componentes: ...
- Riscos com mitigação: ...
- Premissas (evidência e como validar): ...
- Perguntas bloqueantes: ...
- Perguntas não bloqueantes: ...

## 10. Revisão de simplicidade
- Padrões existentes reutilizados: ...
- Abstrações/infra adicionadas e justificativas: ...
- Otimizações evitadas por falta de evidência: ...
- Limpeza e remoções, impacto na compatibilidade (ou `nenhuma remoção`): ...

## 11. Handoff para decomposição
- Contrato estruturado: `planning-contract.yaml`
- Fatias de trabalho sugeridas (sem detalhar tasks): ...
- Interfaces/contratos que devem estabilizar antes do paralelismo: ...
- Áreas que não devem sofrer edição simultânea: ...
- Critério para iniciar decomposição: `status: ready`, sem blockers e contrato consistente com este plano.

## Formato compacto para `light`

Use somente os cinco blocos abaixo; tabelas são opcionais. Eles substituem as onze seções acima. Omita tópicos inaplicáveis sem criar listas de `não aplicável`. Se o risco exigir análise maior, eleve a complexidade.

1. **Identificação e escopo:** origem, slug, tipo, stack, `light` com justificativa, status, objetivo e não objetivos.
2. **Evidências e solução:** comportamento atual/esperado, causa provável e como confirmar (bugs), arquivos/padrões relevantes, busca por reuso se houver componente novo e `CMP-*` com responsabilidades, requisitos associados e dependências reais.
3. **Aceite e verificação:** requisitos `REQ-*` e aceites `AC-*` em Given/When/Then (com as bordas relevantes). Os casos `VER-*` vão para o `tests.md` compacto (regressão, feliz, erro afetado); aqui apenas o resumo da cobertura.
4. **Riscos e pendências:** avaliação de segurança do que mudou (ou ausência de impacto com motivo), premissas com evidência e confirmação, blockers e efeitos em contratos/capacidade/operação apenas quando relevantes.
5. **Limpeza e entrega:** remoções ou `nenhuma remoção`, caminho do contrato e condição para decompor.

O contrato usa `schema_version: 2` e os campos obrigatórios. Uma correção pode conter um requisito, um componente e uma verificação se isso cobrir todo o comportamento; não divida itens apenas para aumentar a quantidade de IDs.
