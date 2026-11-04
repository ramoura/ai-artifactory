# Template de ADR — `tasks/<slug>/adrs/ADR-###.md`

Compartilhado entre `/prd` (decisões de produto) e `/planning` (decisões técnicas). A numeração é **única por história**: pegue o próximo número livre em `adrs/`, com 3 dígitos. Nunca renumere. Uma decisão revertida recebe status `Substituído por ADR-###` e o novo ADR explica o porquê.

```markdown
# ADR-###: [título]

- Status: [Proposto | Aceito | Obsoleto | Substituído por ADR-###]
- Escopo: [produto | técnico]
- Data: AAAA-MM-DD
- Origem: [/prd | /planning] — [pergunta ou evidência que motivou]

## Contexto
[Problema que motiva a decisão, restrições e forças em jogo. Cite evidências do repositório (caminhos) quando houver.]

## Decisão
[O que foi decidido, de forma específica.]

## Alternativas consideradas

### [Alternativa 1]
- Descrição:
- Prós:
- Contras:
- Por que foi descartada:

### [Alternativa 2]
- ...

## Consequências
- Positivas:
- Negativas / trade-offs:
- Riscos e mitigação:

## Rastreabilidade
- Requisitos/aceites afetados: REQ-*, AC-*, RN-*
- Componentes (quando técnico): CMP-*

## Referências
- [documentos, caminhos do repositório, fontes externas consultadas]
```

## Quando escrever um ADR

- **`/prd`:** direção de produto escolhida; exclusões de escopo relevantes; regra de negócio decidida entre alternativas reais.
- **`/planning`:** abordagem técnica principal (sempre ao menos um em `standard`/`deep`); escolha de padrão, tecnologia ou modelo de dados; mudança de contrato; desvio de convenção do repositório.
- Em `light`, ADR só se houver alternativa real descartada. Não crie ADR cerimonial.
