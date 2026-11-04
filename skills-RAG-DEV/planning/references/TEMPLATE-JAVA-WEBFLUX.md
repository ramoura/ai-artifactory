# Template Java + Spring Boot WebFlux

Use **somente** quando evidências do repositório confirmarem Java + WebFlux. Este arquivo cobre o **desenho** (seções do `plan.md`); as regras de escrita e revisão de código estão em `../../implementation/references/stacks/java-spring-webflux.md`. Use o formato completo ou compacto (`light`) do `TEMPLATE-GENERICO.md` e **substitua/complemente** as áreas pertinentes abaixo. Não presuma Redis, SQS, ECS, Datadog ou parceiros externos se eles não estiverem presentes.

## Seção 3 — evidências adicionais
- Java, Spring Boot, Reactor e dependências relevantes com versões encontradas em `pom.xml`/`build.gradle`.
- Controllers/handlers, services/use cases, repositories/adapters e configuração `WebClient` existentes.
- Testes comparáveis (JUnit, Reactor Test, WebTestClient, WireMock/Testcontainers apenas se usados).
- Instrumentação existente: Micrometer, OpenTelemetry, Datadog, logs estruturados, tracing.

## Seção 5 — solução e concorrência
- Fronteira reativa: Mono/Flux, validação, propagação de contexto, tratamento de erros e cancelamento.
- Operações de terceiros: política de timeouts, fallback, retry, idempotência e conexão, se afetadas.
- Nenhum `block()`/I/O síncrono no event loop; se inevitável, explicar isolamento e seus custos.
- Estado: atomicidade, TTL, conexões e concorrência quando houver Redis/banco/cache.

## Seção 6 — contratos
- Rotas, verbos, status, request/response DTO, erros e compatibilidade.
- Integração por eventos/filas apenas quando confirmada: payload, ack, duplicidade, DLQ e retry.
- Migração de banco e contratos com consumidores, se aplicáveis.

## Seção 7 — SLOs e caminho crítico
- Confirmar se o caminho é `hot`; coletar requisitos reais de RPS, p95/p99, erro e disponibilidade.
- Investigar apenas os pools/limites existentes e realmente envolvidos, como connection providers, filas, threads ou backpressure.
- Quando a mudança afetar capacidade, aplicar cálculos, fontes e critérios de bloqueio de `CRITERIOS-ENGENHARIA.md` § 4, incluindo impacto de retries e saturação.
- Planejar métricas/testes sem alegar que foram executados.

## Seção 5.1 — interfaces centrais
- Assinaturas reativas (`Mono<T>`/`Flux<T>`) dos ports/services e a convenção de erro (exceções de domínio, `onErrorMap`, `ProblemDetail` se usado).

## Seção 8 / `tests.md` — validação
- Unidade: lógica, regras e exceções com JUnit/Mockito ou padrão existente.
- Reativo: fluxo, timeout, cancelamento e fallback com `StepVerifier`, se adequado.
- Integração: `WebTestClient`, banco/testcontainers/WireMock **apenas se disponíveis e pertinentes**.
- Carga: cenário, SLO e distribuição de tráfego somente para fluxo crítico com metas verificáveis.

> Para um bug pequeno em um handler, não exigimos todas essas análises. Registre somente o que impacta a correção e as regressões possíveis.
