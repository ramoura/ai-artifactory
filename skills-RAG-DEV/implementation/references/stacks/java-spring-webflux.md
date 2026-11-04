# Stack: Java + Spring Boot WebFlux (reativo)

Precedência e semântica de **default** / **proibição**: ver `README.md` deste diretório. Complementa `planning/references/TEMPLATE-JAVA-WEBFLUX.md` (desenho); este arquivo trata de escrita e revisão de código.

## Detecção

- `spring-boot-starter-webflux`; `Mono`/`Flux` nas assinaturas; Netty; `WebClient`; R2DBC ou drivers reativos de Mongo/Redis.
- Identifique o estilo: anotado (`@RestController`) ou funcional (`RouterFunction`/`HandlerFunction`). Siga o que o repositório usa.
- Registre versões de Spring Boot, Reactor, bibliotecas de teste (`reactor-test`, `WebTestClient`, WireMock/MockWebServer) **a partir dos arquivos**.

## Convenções de código

Defaults:
- A cadeia é reativa de ponta a ponta: handler → serviço → adaptador retornam `Mono<T>`/`Flux<T>`.
- Efeito colateral e código que pode lançar entram em `Mono.fromCallable`/`Mono.defer`/`Flux.defer`, nunca em `Mono.just(chamada())`.
- Biblioteca bloqueante inevitável (JDBC legado, SDK síncrono) fica isolada em `subscribeOn(Schedulers.boundedElastic())`, com o motivo registrado e o ponto de isolamento único.
- Contexto de requisição (correlação, tenant, usuário) propaga pelo Reactor `Context` (`deferContextual`/`contextWrite`), não por `ThreadLocal`.
- `WebClient` é um bean configurado (base URL, timeouts, codecs), nunca criado por requisição.
- `map` para transformação síncrona; `flatMap` para operação assíncrona; `concatMap` quando a ordem importa; `flatMap` com concorrência explícita quando há fan-out.
- Injeção por construtor, DTO separado de entidade, `BigDecimal` para dinheiro e `java.time` para datas, como na stack MVC.

Proibições:
- `block()`, `blockFirst()`, `blockLast()`, `toFuture().get()` ou `Thread.sleep` em qualquer caminho servido pelo event loop.
- `subscribe()` dentro de operador (subscribe aninhado) para disparar trabalho paralelo.
- `ThreadLocal`/MDC direto para contexto de requisição.
- `Mono.just(...)` ou `Mono.error(...)` com argumento que executa I/O ou pode lançar ao ser avaliado.
- Injeção por campo em código novo; entidade de persistência entrando ou saindo pela API; `double`/`float` para dinheiro (mesmas proibições da stack MVC).

## Erros e validação

Defaults:
- Validação de entrada com Bean Validation (`@Valid` no corpo) ou `Validator` explícito em handler funcional, conforme o repositório.
- Tratamento central em `@RestControllerAdvice` ou `WebExceptionHandler`, no formato de erro do repositório (`ProblemDetail` se não houver outro).
- Erro de infraestrutura traduzido para exceção de domínio com `onErrorMap(TipoEspecifico.class, ...)`.
- Ausência de valor: `switchIfEmpty(Mono.defer(() -> Mono.error(new XNotFoundException(...))))`; o `defer` evita avaliação antecipada.
- Timeout explícito em chamada externa (`.timeout(Duration, fallback)` ou configuração do `WebClient`) e retry limitado (`Retry.backoff(n, d).filter(...)`) só em operação idempotente.

Proibições:
- `onErrorResume(e -> Mono.empty())` ou `onErrorReturn` sem filtrar o tipo da exceção.
- Engolir erro em `doOnError` e seguir como sucesso.
- Retry sem `maxAttempts` ou em operação não idempotente.
- `switchIfEmpty(Mono.error(...))` sem `defer` quando a construção do erro tem custo ou efeito.

## Testes

Defaults:
- `StepVerifier` para `Mono`/`Flux`: `expectNext`/`assertNext` + `verifyComplete()`, ou `expectError(Tipo.class)`/`verifyErrorSatisfies`.
- `StepVerifier.withVirtualTime` para timeout, retry e delay; nunca esperar tempo real.
- Endpoint: `@WebFluxTest` + `WebTestClient`, verificando status e corpo; integração HTTP externa com WireMock/MockWebServer **se já presentes**.
- Teste de cancelamento (`thenCancel()`) quando a task toca streaming ou recursos abertos.
- Nome do teste descreve comportamento e cita o `VER-###`.
- Mock só na fronteira de I/O; não mocke `Mono`/`Flux` retornando `Mono.just` de forma que esconda o caminho de erro: inclua caso com `Mono.error`.

Proibições:
- `.block()` em teste como substituto de `StepVerifier` quando o repositório já usa `reactor-test`.
- `Thread.sleep` ou `await` baseado em relógio real.
- Teste que só verifica `verifyComplete()` sem asserção sobre o valor.

## Segurança

- Segurança reativa (`ReactiveSecurityContextHolder`, `SecurityWebFilterChain`); endpoint novo nunca fica fora da cadeia por omissão.
- Autorização por recurso com o principal do contexto, nunca com id vindo do cliente sem conferência.
- URL de `WebClient` montada com entrada do usuário exige lista de destinos permitidos (SSRF).
- Sem segredos em configuração versionada; logs sem PII nem corpo completo; não logar `Context` inteiro.
- Consultas R2DBC/Mongo parametrizadas; nenhuma concatenação.

## Desempenho e recursos

- Backpressure respeitado: em fonte sem controle (fila, WebSocket), usar `onBackpressureBuffer` com limite ou `limitRate`.
- `collectList()`/`buffer()` só em coleção com tamanho limitado conhecido; stream aberto permanece `Flux`.
- `flatMap` com fan-out tem concorrência explícita compatível com o pool do dependente; `parallel()` só com evidência de CPU-bound.
- Timeouts de conexão, resposta e leitura configurados no `WebClient`; pool do `ConnectionProvider` não alterado sem medição.
- Nada de I/O síncrono (arquivo, JDBC, `RestTemplate`) no event loop; se existir legado, registrar.
- `@Transactional` com R2DBC exige `ReactiveTransactionManager`; verificar que o gerenciador em uso é reativo.

## Armadilhas frequentes

- Nada acontece sem subscriber: cadeia montada e não retornada não executa.
- Assinar a mesma cadeia duas vezes repete o efeito (`cache()` só com intenção clara).
- `doOnNext` com I/O: efeito colateral bloqueante disfarçado.
- `Optional` dentro de cadeia reativa: preferir `Mono.empty()` + `switchIfEmpty`.
- `Mono<Void>` que perde o erro por `then()` encadeado antes do tratamento.
- Misturar repositório JPA (bloqueante) em serviço reativo sem isolamento.
- `MDC` populado em um thread e lido em outro: usar `Context` + hook do projeto.
- `zip` com um `Mono` vazio cancela os demais e completa vazio sem erro.

## Checagens para review

- [ ] Nenhum `block*()`, `Thread.sleep` ou I/O síncrono no caminho do event loop; bloqueio inevitável isolado em `boundedElastic` com motivo.
- [ ] Nenhum `subscribe()` aninhado; nenhum `Mono.just` com chamada que faz I/O ou lança.
- [ ] Contexto de requisição via Reactor `Context`, não `ThreadLocal`.
- [ ] `onErrorResume`/`onErrorReturn` filtram o tipo; `switchIfEmpty` com `defer` quando constrói erro.
- [ ] Chamada externa nova tem timeout; retry limitado e idempotente.
- [ ] Fan-out com concorrência explícita; nenhum `collectList` sobre fonte ilimitada.
- [ ] `WebClient` como bean configurado; URL com entrada do usuário validada contra lista.
- [ ] Cada `VER-*` da task existe com `StepVerifier`/`WebTestClient`, inclui caminho de erro e usa tempo virtual onde há delay/timeout.
- [ ] Nenhum `.block()` em teste novo onde `reactor-test` está disponível.
- [ ] Transação reativa com gerenciador reativo; nenhum repositório bloqueante introduzido.
- [ ] Injeção por construtor; nenhuma entidade de persistência na API; dinheiro em `BigDecimal`.
