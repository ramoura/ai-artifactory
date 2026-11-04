# Template de `tasks/<slug>/user-stories.md`

Catálogo canônico de comportamento da história. Toda story, critério de aceite e caso de borda vive **aqui e só aqui**; o `prd.md` apenas indexa este arquivo.

Consumidores:
- a `/planning`, que preserva os IDs, mapeia cada `REQ-*` para componentes `CMP-*` e cobre cada `AC-*` com verificações `VER-*` concretas em `tests.md`;
- a `task-breakdown`, a `task-review` e a `integration-validation`, que validam a implementação contra estes critérios.

## Regras de ID

- **Stories:** `REQ-###`. Uma story é um requisito do contrato.
- **Critérios e casos de borda:** `AC-###`, numeração **global e sequencial** (não reinicia por story). Cada AC indica seu tipo: `criterion` ou `edge_case`.
- **Regras de negócio:** `RN-###`, definidas no `prd.md` e citadas nos AC que as exercitam.
- **IDs são permanentes:** nunca renumere nem reutilize. Item descartado fica marcado `(withdrawn)` no lugar.
- A `/planning` pode acrescentar `REQ-*`/`AC-*` técnicos (por exemplo, compatibilidade, migração) continuando a numeração, **somente** na seção `## AC técnicos (planning)` ao final deste arquivo. Ela não edita as stories, critérios e bordas escritos pela `/prd`.

## Esqueleto

```markdown
# User stories — [nome da feature]

Catálogo canônico de comportamento. Complementa `prd.md`; consumido por `plan.md`,
`tests.md` e `planning-contract.yaml`.

## Personas

- **[Persona]** — [quem é, contexto, o que precisa desta feature]

## Índice

| REQ | Área | Persona | Story | Status |
|---|---|---|---|---|
| REQ-001 | [área] | [persona] | [resumo em uma linha] | active |

## [Área 1]

### REQ-001: [título curto]

**Como** [persona], **quero** [capacidade], **para** [resultado].

Regras aplicáveis: RN-001, RN-003

Critérios de aceite:

- **AC-001** (criterion): **Dado** [contexto inicial], **quando** [ação], **então** [resultado observável].
- **AC-002** (criterion): **Dado** [...], **quando** [...], **então** [...].

Casos de borda:

- **AC-003** (edge_case · permissions): **Dado** [usuário sem papel X], **quando** [ação], **então** [comportamento esperado observado].
- **AC-004** (edge_case · repetition): **Dado** [envio já concluído], **quando** [reenvio idêntico], **então** [comportamento idempotente esperado].

## AC técnicos (planning)

Seção escrita **apenas pela `/planning`**, criada só quando houver requisito ou critério técnico que o PRD não cobre (compatibilidade, migração, contrato com consumidor existente). Continua a numeração; cita o `plan.md` que o justifica. A `/prd` em modo atualização preserva esta seção intacta.

### REQ-0NN: [título técnico] — origem: `plan.md` § [seção]

- **AC-0NN** (criterion): **Dado** [...], **quando** [...], **então** [...].
```

## Varredura de casos de borda

Teste **toda story contra toda classe** e registre cada achado como um `AC` do tipo `edge_case`, com a classe e o comportamento esperado. Pule uma classe só depois de testá-la de fato: a maioria dos "não se aplica" está errada, e classe não varrida é por onde comportamento não tratado chega à produção.

| Classe (`edge_class`) | O que testar |
| --- | --- |
| `invalid_input` | Malformado, tipo errado, fora da faixa, não parseável, hostil |
| `empty` | Coleção vazia, string em branco, dado opcional ausente, primeiro uso |
| `limits` | Tamanho máximo, cota, truncamento, limite de paginação, rate limit |
| `permissions` | Não autorizado, sessão expirada, papel insuficiente, acesso entre tenants |
| `concurrency` | Mesma ação duas vezes em paralelo, dois atores no mesmo recurso, leitura obsoleta |
| `interruption` | Cancelar no meio, queda de conexão, reinício de processo, conclusão parcial |
| `repetition` | Retry após sucesso, envio duplicado, replay: a ação é idempotente? |
| `ordering` | Passos fora de ordem, pré-requisito pulado, navegação para trás, deep link |
| `state` | Ação sobre entidade excluída/fechada/arquivada, salto de estado inválido |
| `scale` | Comportamento com zero itens, volume típico e 100× o típico |

**Proporcionalidade:**
- Em **features** (`standard`/`deep`), a varredura é completa.
- Se o comportamento esperado de uma borda **não puder ser decidido** pelo código, pelas regras ou pelas respostas do usuário, ele vira uma pergunta (passo 3) ou uma Questão em aberto. Nunca invente.

## Regras de escrita

- Descreva **comportamento que o usuário observa**, nunca implementação ("vê o último rascunho salvo", não "lê da tabela drafts").
- **Uma story por capacidade.** Separar mantém os critérios testáveis; juntar para encurtar o catálogo esconde comportamento.
- **Todo AC precisa ser verificável no produto entregue:** alguém consegue marcá-lo como verdadeiro ou falso usando a funcionalidade.
- **Todo caso de borda traz condição e comportamento esperado** ("upload acima do limite → rejeitado com mensagem de limite"), nunca só "uploads grandes".
- **Personas secundárias** (admin, operador, integrador) têm stories próprias: a maior parte das bordas não tratadas vive nos fluxos delas.
- **Valores exatos** (limites, prazos, defaults) vêm das regras `RN-*`. Se não existirem, viram Questão em aberto.
