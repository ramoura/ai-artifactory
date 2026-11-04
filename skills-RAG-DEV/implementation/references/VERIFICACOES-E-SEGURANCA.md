# Validações proporcionais ao trabalho

Esta referência é um guia de **aplicabilidade**, não um checklist universal e não define métricas arbitrárias. Regras da Planning v2, da história e do projeto prevalecem.

## Por tipo de demanda

### Bug localizado
- Confirmar causa ou comportamento reproduzível, se houver ambiente de teste.
- Adicionar teste de regressão representativo quando viável.
- Limitar alteração ao problema e aos casos adjacentes relevantes.

### Feature
- Cobrir cenários positivos, negativos, regras de negócio, erros e bordas definidos nos `AC-*`.
- Conferir contratos, persistência e compatibilidade com consumidores existentes.
- Verificar feature flags, rollout e rollback somente quando exigidos.

### Mudança em fluxo crítico ou sensível
- Validar concorrência, idempotência, timeouts, retries, recursos e observabilidade de acordo com a stack real.
- Não inventar metas de throughput ou cobertura; usar valores com origem verificável.
- Respeitar aprovações para auth, antifraude, PII, segredos, dados financeiros e migrações destrutivas.

## Por stack

Convenções de código, testes, segurança e armadilhas de Java + Spring MVC, Java + Spring WebFlux, Node backend e React estão em `stacks/<stack>.md`, com a regra de precedência em `stacks/README.md`. Carregue o arquivo da stack identificada no repositório; use comandos (`./mvnw`, `./gradlew`, `npm test`, `pnpm test`) apenas se existirem e forem permitidos.

## Segurança e ambiente

- Não incluir tokens, senhas, URLs internas sensíveis, payloads pessoais ou conteúdo confidencial nas evidências.
- Não executar comandos destrutivos (apagamento amplo, reset de banco, deploy, alteração em produção) por conta própria.
- Não baixar código externo, instalar ferramenta ou conectar rede corporativa sem permissão.
- Não silenciar falhas de teste para obter saída verde; documentar causa e consequência.

## Reporting

Cada verificação precisa indicar: `VER-*`, comando real (ou `null`), `passed | failed | not_run`, evidência resumida e motivo quando não executada. Critério de aceite sem validação real deve ser `partial` ou `not_verified`, não `verified`.
