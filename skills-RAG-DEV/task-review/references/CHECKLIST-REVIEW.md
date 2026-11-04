# Checklist de revisão proporcional

Use como guia, **não como quota**. Inspecione apenas partes relevantes da task, contrato e arquivos modificados. Regras corporativas do repositório prevalecem.

## Funcional e integração

- Requisito/critério de aceite tem comportamento observável em produção e teste?
- Entradas inválidas, limites, erros, nulidade e retorno preservam o contrato?
- Mudanças de schema/API/evento têm consumidores, versão e compatibilidade verificados?
- Há comportamento regressivo em fluxos adjacentes? Testes testam o efeito, não só a implementação?

## Segurança e privacidade

- Autorização por recurso/ação, validação e sanitização pertinentes?
- Dados sensíveis/segredos em logs, exceções, diff, fixtures ou artefatos?
- Injeção, path traversal, SSRF, deserialização, permissões e vazamento de informações, se houver superfície correspondente?
- Política e aprovação exigidas para autenticação, fraude, pagamentos e migrações?

## Resiliência, concorrência e recursos

- Timeouts, retries limitados, idempotência, cancelamento e degradação previsível?
- Corridas, acesso concorrente, transação, consistência e fechamento de recursos?
- Itens específicos de Spring MVC, WebFlux, Node e React: seção **Checagens para review** de `../../implementation/references/stacks/<stack>.md`.
- Meta de p95/RPS/erro tem fonte real? Nunca exigir números universais por padrão.

## Testes e evidência

- Cada `VER-*` possui evidência observável, não apenas alegação?
- Falhas de teste atribuídas corretamente (nova mudança, preexistente, ambiente)?
- Testes relevantes são determinísticos, com asserções úteis e dados isolados?
- Lint/build/teste realmente executados? Se não, impacto e motivo declarados?
- Existe regressão em integração da história que requer `integration-validation`?

## Severidade orientativa

- `critical`: exploração ou ruptura grave e demonstrável, risco inaceitável antes da entrega.
- `high`: bug relevante em fluxo principal, segurança ou contrato com impacto claro.
- `medium`: defeito funcional parcial, falha de borda relevante, regressão ou lacuna de teste essencial corrigível.
- `low`: risco/qualidade menor, não necessariamente bloqueante.

**Não classifique sem evidência.** Vulnerabilidade suspeita sem reprodução deve ser descrita com nível de confiança e condição de confirmação; não afirmar exploração comprovada.
