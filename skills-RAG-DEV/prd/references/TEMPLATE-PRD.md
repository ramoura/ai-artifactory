# Template de `tasks/<slug>/prd.md`

O PRD é lido por agentes (`planning`, `task-breakdown`, `task-review`, `integration-validation`). Ele existe para entregar **regras de negócio, comportamento de domínio e intenção de produto**.

KPIs, metas de negócio, cronograma e fases de rollout não têm consumidor nesta pipeline: deixe-os fora, a menos que a história traga uma exigência explícita.

Preencha cada seção com o resultado do interrogatório. Onde faltar informação, registre em **Questões em aberto**; não chute.

```markdown
# PRD — [nome da feature]

- Slug: [slug]
- Origem: `tasks/<slug>/historia.md`
- Status: [ready | blocked] — [motivo]
- Data: AAAA-MM-DD

## 1. Visão geral
- Problema que resolve:
- Para quem:
- Por que tem valor:

## 2. Objetivos
Resultados de produto como **comportamento observável**, não métricas:
- O que o usuário passa a conseguir fazer
- O que o sistema passa a garantir ou impor
- O que deixa de ser necessário, vira automático ou passa a ser impossível

## 3. User stories
Índice de `user-stories.md`. Não repita as stories aqui.
- [Área]: REQ-001 a REQ-004 — [tema]
- Catálogo completo: [user-stories.md](user-stories.md)

## 4. Funcionalidades
Para cada funcionalidade: o que faz, por que importa, comportamento em alto nível, requisitos funcionais (`REQ-*`) e interação com as demais.

## 5. Regras de negócio
Regras que a implementação precisa impor, com precisão. Cada uma tem ID `RN-###`:
- **RN-001 (invariante):** [ex.: "uma despesa pertence a exatamente um centro de custo"]
- **RN-002 (validação):** [regra] → [resultado visível ao usuário]
- **RN-003 (permissão/visibilidade):** [regra por persona]
- **RN-004 (ciclo de vida):** [estados existentes, transições permitidas e quando]
- **RN-005 (cálculo/limite/default):** [fórmula ou valor **exato**]

## 6. Experiência do usuário
- Personas e objetivos
- Fluxos principais, passo a passo
- UI/UX e acessibilidade (quando houver interface)
- Descoberta e primeiro uso

## 7. Restrições de alto nível
Limites que moldam o produto **sem prescrever implementação**:
- Integrações obrigatórias com sistemas existentes
- Conformidade e regulação
- Desempenho percebido pelo usuário, **somente com fonte** (nunca inventar números)
- Privacidade e segurança de dados

Banco, framework, desenho de API e padrões de arquitetura pertencem à `plan.md`.

## 8. Fora de escopo
Capacidades que o usuário **decidiu** não incluir, com o motivo. Exclusões registram decisão do usuário, nunca gestão de tamanho.

## 9. Decisões (ADRs)
- [ADR-001: título](adrs/ADR-001.md) — resumo em uma linha

## 10. Questões em aberto
- [Questão] — impacto: [bloqueia | não bloqueia] — [quem decide]
```

## Regras

- `Status: blocked` quando uma questão em aberto mudar regra de negócio, permissão ou fluxo principal. A `/planning` não marca `ready` sobre um PRD `blocked`.
- Toda `RN-*` é exercitada por pelo menos um `AC-*` em `user-stories.md`.
- Nada de requisito técnico disfarçado: se a frase cita tabela, endpoint, fila ou classe, ela não pertence ao PRD.
