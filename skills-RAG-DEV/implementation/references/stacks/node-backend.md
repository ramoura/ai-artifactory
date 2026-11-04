# Stack: Node.js backend (TypeScript ou JavaScript)

Precedência e semântica de **default** / **proibição**: ver `README.md` deste diretório.

## Detecção

- `package.json` com Express, Fastify, NestJS, Koa ou Hono e sem `react`.
- Registre a partir dos arquivos: TypeScript ou JavaScript e o `strict` do `tsconfig.json`; sistema de módulos (`"type": "module"` ou CommonJS); runner de testes (Vitest, Jest, `node:test`); ORM/driver (Prisma, Drizzle, TypeORM, Knex, `pg`); biblioteca de validação (zod, joi, class-validator, ajv/Fastify schema); logger; lint e formatter.
- Versão do Node em `engines`, `.nvmrc` ou Dockerfile.

## Convenções de código

Defaults:
- TypeScript `strict`; tipo explícito no retorno de funções exportadas; `unknown` + narrowing no lugar de `any`.
- `async/await` em vez de cadeias `.then`; toda promise é aguardada, retornada ou explicitamente descartada com `void` e comentário do motivo.
- Camadas: rota/controller → serviço → repositório. Handler de rota não contém regra de negócio nem SQL.
- Entrada validada na fronteira (corpo, query, params, headers) com a biblioteca de schema do repositório; o tipo interno deriva do schema.
- Configuração lida e validada uma vez na inicialização (módulo `config`), nunca `process.env.X` espalhado; variável ausente falha o boot.
- Erros são subclasses de `Error` com `code` estável; nunca `throw "string"` nem objeto solto.
- Dependências injetadas explicitamente (parâmetros, fábrica ou o container do framework, como no NestJS), para que o teste substitua a fronteira de I/O.
- Dinheiro em inteiro de centavos ou biblioteca decimal; datas em `Date`/`Temporal` com fuso explícito; nunca aritmética de ponto flutuante para valores monetários.
- Estilo de export (named/default), nomenclatura de arquivos e organização de pastas seguem o repositório.

Proibições:
- `any` em código novo (inclusive `as any`); `@ts-ignore`/`@ts-expect-error` sem motivo escrito.
- Promise flutuante (chamada assíncrona sem `await`, `return` ou `void` justificado).
- Regra de negócio ou consulta dentro do handler de rota.
- `process.env` lido fora do módulo de configuração.

## Erros e validação

Defaults:
- Middleware/filtro central de erro traduz exceção de domínio para status e corpo no formato do repositório: 400 validação, 401/403 autenticação/autorização, 404 ausente, 409 conflito, 500 só para o inesperado, com log.
- Handlers assíncronos no Express 4 passam por wrapper que encaminha rejeição ao `next` (Express 5 e Fastify fazem isso nativamente).
- `process.on('unhandledRejection')` e `uncaughtException` registram e **encerram** o processo; o orquestrador de containers reinicia.
- Chamada HTTP de saída com `AbortSignal.timeout`/timeout do cliente; retry limitado, com backoff, só em operação idempotente.
- Encerramento gracioso em `SIGTERM`: parar de aceitar conexões, drenar as em curso, fechar pool e sair.

Proibições:
- `catch (e) {}` vazio ou que só loga e devolve 200.
- Stack trace ou mensagem de erro de driver na resposta HTTP.
- Processo que continua após `uncaughtException` sem reinício.
- `try { await x } catch` onde o `await` foi esquecido e o erro escapa do bloco.

## Testes

Defaults:
- O runner do repositório. Unitário com mocks apenas na fronteira de I/O (repositório, cliente HTTP, relógio, fila); não mocke módulos internos em bloco.
- HTTP: `supertest` ou `app.inject()` (Fastify) contra a aplicação montada, checando status e corpo.
- Banco em teste de integração: transação revertida, banco descartável ou Testcontainers **se já presentes**; nunca o banco de desenvolvimento compartilhado.
- Tempo controlado com fake timers; nenhum `setTimeout` real para sincronizar.
- Sem rede real em teste unitário; integração externa via MSW/nock/servidor fake **se já presentes**.
- Nome do teste descreve comportamento e cita o `VER-###`; um comportamento por teste; dados próprios por teste.

Proibições:
- `.only`/`.skip` versionado; asserção removida ou afrouxada para deixar a suíte verde.
- Snapshot para lógica de negócio (aceitável só para saída estática).
- Teste que depende da ordem de execução ou de estado deixado por outro teste.

## Segurança

- Headers de proteção pelo mecanismo do framework (`helmet` ou equivalente **se já presente**); CORS explícito por origem, nunca `*` com credenciais.
- Consultas parametrizadas ou ORM; nenhuma interpolação de entrada em SQL, comando de shell (`child_process`) ou `eval`/`new Function`.
- Operação com caminho de arquivo: `path.resolve` + verificação de que o resultado permanece sob o diretório permitido.
- JWT: algoritmo fixado na verificação, expiração obrigatória, segredo/chave fora do código.
- Rate limit em autenticação e endpoints sensíveis, conforme a infraestrutura do projeto.
- Limite de tamanho de corpo configurado; merge de objeto com entrada do usuário só em propriedades conhecidas (prototype pollution).
- Logs sem senha, token, PII ou corpo completo.
- Nenhuma dependência nova nem alteração de lockfile fora do escopo autorizado.

## Desempenho e recursos

- Nada de API síncrona de I/O ou criptografia (`readFileSync`, `pbkdf2Sync`, `execSync`) no caminho de requisição.
- Operações independentes de I/O em `Promise.all`/`allSettled`; laço `for await` só quando a ordem ou o limite de concorrência importa.
- CPU intensiva (parse grande, hash, compressão) em `worker_threads` ou fora do processo; o event loop não bloqueia.
- Respostas e uploads grandes em stream; listagens paginadas com limite máximo.
- Pool de conexões único por processo; conexões fechadas no encerramento.
- Cache em memória só com limite de tamanho e TTL; nunca como fonte de verdade entre instâncias.

## Armadilhas frequentes

- `await` dentro de `forEach`: não espera. Use `for...of` ou `Promise.all`.
- Variável de ambiente é sempre string: `"false"` é truthy.
- Comparação com `==`; `parseInt` sem base; `JSON.parse` sem `try`.
- Import circular que deixa objeto `undefined` na inicialização.
- Mutação de `req`/`ctx` para passar dados entre camadas em vez de parâmetro tipado.
- Fuso horário do servidor influenciando `new Date()`; armazenar e transportar em UTC/ISO 8601.
- Confiar na ordem de chaves de objeto.
- `process.exit` em código de biblioteca ou dentro de handler.

## Checagens para review

- [ ] Nenhum `any`, `@ts-ignore` sem motivo ou promise flutuante no diff.
- [ ] Entrada validada por schema na fronteira; handler sem regra de negócio nem SQL.
- [ ] Erro central no formato do repositório; nenhum `catch` que engole; handler assíncrono encaminha rejeição.
- [ ] Chamada externa nova tem timeout; retry limitado e idempotente.
- [ ] Nenhuma API síncrona de I/O/cripto no caminho de requisição; CPU pesada fora do event loop.
- [ ] Consulta parametrizada; nenhum `exec`/`eval` com entrada; caminho de arquivo confinado.
- [ ] Configuração via módulo validado; nenhum segredo, PII ou corpo completo em log.
- [ ] Nenhuma dependência nova ou mudança de lockfile fora do escopo.
- [ ] Cada `VER-*` da task existe no runner do repositório, com alvo, entrada e saída de `tests.md`, mocks só na fronteira de I/O e sem rede real.
- [ ] Nenhum `.only`, `.skip` novo ou asserção enfraquecida.
