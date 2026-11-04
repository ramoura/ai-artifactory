# Stack: Java + Spring Boot MVC (servlet)

Precedência e semântica de **default** / **proibição**: ver `README.md` deste diretório.

## Detecção

- `pom.xml`/`build.gradle` com `spring-boot-starter-web` e **sem** `spring-boot-starter-webflux`.
- `@RestController`/`@Controller` com retornos síncronos; `HttpServletRequest`; Tomcat ou Jetty.
- Registre versão de Java, Spring Boot, JPA/JDBC, mapeador (MapStruct, manual), Lombok, validação e framework de testes **a partir dos arquivos**, não por suposição.

## Convenções de código

Defaults:
- Injeção **por construtor**, campos `final`; uma única construtora dispensa `@Autowired`.
- Controller só traduz HTTP: valida entrada, chama um serviço, monta resposta. Regra de negócio fica no serviço ou domínio.
- DTO de request/response separado da entidade; `record` quando a versão de Java permitir. Entidade JPA nunca é retornada pela API nem recebida como `@RequestBody`.
- `@Transactional` no serviço, método público, o mais curto possível; leitura com `readOnly = true`.
- `java.time` para datas; `BigDecimal` para dinheiro; `Optional` só como retorno, nunca como campo ou parâmetro.
- Organização de pacotes (por feature ou por camada), nomenclatura e uso de Lombok seguem o que o repositório já faz.

Proibições:
- Injeção por campo em código novo.
- Lógica de negócio em controller ou em `@ControllerAdvice`.
- `double`/`float` para valores monetários.
- Estado mutável em `static` ou em beans singleton.

## Erros e validação

Defaults:
- Validação de entrada com Jakarta Bean Validation (`@Valid` no `@RequestBody`/`@PathVariable`), regras de domínio no serviço.
- Tratamento central em `@RestControllerAdvice`, respondendo no formato de erro que o repositório já usa (`ProblemDetail` no Spring 6+ se não houver outro).
- Exceções de domínio específicas (`XNotFoundException`, `XConflictException`) mapeadas para status: 400 validação, 404 ausente, 409 conflito de estado, 422 se o repositório adota, 500 nunca intencional.
- Toda resposta de erro tem `code` estável e mensagem sem dados internos.

Proibições:
- `catch (Exception e)` que engole ou só loga e segue.
- Stack trace, SQL ou nome de classe interna na resposta HTTP.
- Validar no controller com `if` manual o que a anotação de validação já cobre.

## Testes

Defaults:
- JUnit 5 + AssertJ + Mockito, ou o que o repositório usa.
- Controller: `@WebMvcTest` + `MockMvc`, com serviço mockado; checa status **e** corpo (JSONPath).
- Repositório: `@DataJpaTest`; banco real via Testcontainers **apenas se já presente**.
- `@SpringBootTest` só para integração de ponta a ponta; não para testar uma classe.
- Nome do teste descreve comportamento (`cria_despesa_quando_valida`, ou a convenção do repositório) e cita o `VER-###` no nome ou em comentário.
- Um comportamento por teste; dados de teste próprios, sem estado compartilhado entre testes.
- Mock só na fronteira de I/O (repositório, cliente HTTP, relógio). Não mocke a classe sob teste nem valores.

Proibições:
- Teste de getter/setter/construtor/biblioteca de terceiros para inflar cobertura.
- `Thread.sleep` para sincronizar.
- `@MockBean` em `@SpringBootTest` como substituto de teste unitário.
- `@Disabled` ou asserção removida para deixar a suíte verde.

## Segurança

- Autorização por recurso: o id do dono vem do principal autenticado, nunca do corpo ou do path sem conferência.
- `@PreAuthorize`/configuração de `SecurityFilterChain` conforme o repositório; endpoint novo nunca fica fora da cadeia por omissão.
- Consultas parametrizadas (JPQL/Criteria/`@Query` com parâmetros). Concatenação de string em consulta é proibida.
- Sem segredos em `application*.yml` versionado; valores vêm de variável de ambiente ou cofre do projeto.
- Logs sem PII, token ou corpo completo de request.
- Actuator exposto só nos endpoints que o repositório já libera.
- Upload e download: validar tipo e tamanho; caminho de arquivo nunca montado com entrada do usuário sem normalizar e conferir prefixo.

## Desempenho e recursos

- Consultas N+1: `JOIN FETCH` ou `@EntityGraph` quando a relação é usada; verificar com log de SQL em teste quando a mudança toca listagem.
- Listagem paginada (`Pageable`) em endpoint que retorna coleção aberta.
- `spring.jpa.open-in-view=false` é o default para projeto novo; em projeto existente, seguir o que está configurado e registrar.
- Cliente HTTP (`RestClient`/`RestTemplate`/Feign) com timeout de conexão e leitura explícitos; retry limitado e só em operação idempotente.
- `@Async` apenas com executor configurado no projeto.
- Pool de conexões (Hikari) não é alterado sem medição.

## Armadilhas frequentes

- `@Transactional` em método privado ou chamado pela própria classe: não aplica (proxy).
- `LazyInitializationException` ao serializar entidade fora da transação: sintoma de entidade vazando para a API.
- `equals`/`hashCode` em entidade usando id gerado antes de persistir.
- `Optional.get()` sem verificação; preferir `orElseThrow` com exceção de domínio.
- Conversão de fuso: `LocalDateTime` onde o requisito pede instante (`Instant`/`OffsetDateTime`).
- Binding direto de entidade em `@RequestBody` (mass assignment).

## Checagens para review

- [ ] Código novo usa injeção por construtor e não tem lógica de negócio em controller.
- [ ] Entrada validada por Bean Validation; erro central no formato do repositório; nenhum `catch` genérico que engole.
- [ ] Nenhuma entidade JPA entra ou sai pela API.
- [ ] Dinheiro em `BigDecimal`; datas em `java.time` com o tipo coerente com o requisito.
- [ ] Consultas parametrizadas; autorização por recurso presente em endpoint novo.
- [ ] Listagem nova é paginada ou tem limite justificado; relação usada não gera N+1.
- [ ] Cliente HTTP novo tem timeouts; retry, se houver, é limitado e idempotente.
- [ ] Cada `VER-*` da task existe, com o alvo, a entrada e a saída de `tests.md`, no nível certo (`@WebMvcTest`, `@DataJpaTest`, unitário).
- [ ] Nenhum `Thread.sleep`, `@Disabled` novo ou asserção enfraquecida.
- [ ] Nenhum segredo ou PII em código, configuração versionada ou log.
