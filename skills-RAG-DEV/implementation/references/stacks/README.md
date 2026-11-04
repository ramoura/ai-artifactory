# Referências por stack

Um arquivo por stack, lido por três skills:
- `/planning` (desenho): seções **Convenções**, **Erros e validação**, **Testes** e **Desempenho** orientam `plan.md` e `tests.md`.
- `/implementation` (escrita): todas as seções.
- `/task-review` (checagem): a seção final **Checagens para review** de cada arquivo, mais as proibições.

| Arquivo | Quando carregar |
| --- | --- |
| `java-spring-mvc.md` | `spring-boot-starter-web` sem `webflux`; controllers servlet, Tomcat/Jetty |
| `java-spring-webflux.md` | `spring-boot-starter-webflux`, `Mono`/`Flux`, Netty, R2DBC |
| `node-backend.md` | `package.json` com Express, Fastify, NestJS, Koa ou Hono, sem React |
| `react.md` | `react` em `package.json`: Vite, Next.js, Remix ou CRA |
| `playwright-e2e.md` | Sempre que a task tiver `VER-*` do tipo `e2e` em interface web (junto com o arquivo da stack do front) |

Um repositório pode ter mais de uma stack (monorepo, BFF). Carregue só os arquivos das partes que a task toca.

## Precedência (vale para todos os arquivos)

1. Convenções já presentes no repositório (código análogo, testes existentes, lint configurado).
2. `CLAUDE.md`, `AGENTS.md`, `.claude/rules/` e documentação do projeto.
3. Este diretório.

Cada arquivo separa **defaults** de **proibições**:
- **Default**: o que fazer quando o repositório não decide. Nunca justifica refatorar código existente que faz diferente; registre a divergência em `result.md` e siga o repositório.
- **Proibição**: vira finding na review mesmo que o repositório já faça assim. O implementador não introduz código novo com o padrão proibido; código preexistente vai para "Riscos observados fora de escopo".

A ordem de precedência acima governa os **defaults**. Uma proibição só é afastada se `CLAUDE.md`/`AGENTS.md` documentar a exceção **explicitamente, com motivo**; nesse caso o implementador segue o repositório e registra em `result.md`, e o reviewer registra em "Riscos observados" em vez de finding. Convenção implícita (o código só faz assim) não afasta proibição.

Os itens das seções **Segurança** de cada arquivo são todos **proibições**, salvo quando o próprio item disser "default".

Quando a referência cita uma biblioteca (Testcontainers, MSW, zod), ela só vale se a biblioteca **já estiver no repositório**. Nenhuma dependência nova sem autorização.
