# Stack: React (front-end)

Precedência e semântica de **default** / **proibição**: ver `README.md` deste diretório.

## Detecção

- `react` em `package.json`. Registre a partir dos arquivos: versão do React; framework (Vite SPA, Next.js com App ou Pages Router, Remix, CRA legado); TypeScript; roteador; estado global (Redux Toolkit, Zustand, Context); dados de servidor (TanStack Query, SWR, RTK Query, loaders do framework, `fetch` manual); formulários (react-hook-form, Formik, controlado manual); estilo (CSS Modules, Tailwind, styled-components, CSS-in-JS); testes (Vitest ou Jest + Testing Library, MSW, Playwright/Cypress); `eslint-plugin-react-hooks` e `jsx-a11y`.
- Em Next.js, identifique se o componente alterado é servidor ou cliente (`"use client"`) antes de usar hook ou API do navegador.

## Convenções de código

Defaults:
- Componentes de função com hooks; props tipadas por `interface`/`type`, `children` com `React.ReactNode`.
- Estado derivado é calculado no render (ou `useMemo` se caro), nunca copiado para outro estado via `useEffect`.
- `useEffect` só para sincronizar com algo externo ao React (assinatura, DOM imperativo, timer, biblioteca de terceiros). Reação a clique vai no handler; reação a mudança de prop vira cálculo ou `key`.
- Estado de servidor (dados de API) vive na biblioteca de dados do repositório, não em `useState` + `useEffect` manual nem no store global.
- Estado local fica no componente que o usa; sobe só até o ancestral comum que precisa dele. Prop drilling acima de dois ou três níveis vira composição ou contexto.
- Lógica reutilizável em hook `useXxx`; JSX sem regra de negócio inline.
- `key` estável vinda dos dados (id). Índice só em lista estática que nunca reordena, filtra ou insere.
- Formulário: controlado ou com a biblioteca do repositório, nunca misturado no mesmo campo.
- Acessibilidade: elemento semântico (`button`, `a`, `nav`, `label`), `label` ligado ao input, `alt` em imagem, foco e teclado funcionando.
- Handlers nomeados `handleX`; props de evento `onX`.
- Estrutura de pastas, exports e nomenclatura de arquivos seguem o repositório.

Proibições:
- Componente de classe novo.
- Hook chamado condicionalmente, em laço ou depois de `return` antecipado.
- Regra `react-hooks/exhaustive-deps` desabilitada sem comentário que explique por que a dependência omitida é segura.
- Mutação direta de estado (`state.items.push(...)`, `obj.x = ...` em objeto do estado).
- `key` por índice em lista dinâmica.
- `div`/`span` com `onClick` onde existe elemento nativo (`button`, `a`): sem foco, sem teclado, sem papel acessível.

## Erros e estados assíncronos

Defaults:
- Toda visão assíncrona trata os estados de carregamento, erro e vazio explicitamente; nenhum deles fica implícito como "tela em branco".
- Error boundary em nível de rota ou feature, com fallback que permite recuperar.
- Mensagem de erro para o usuário vem de um mapeamento (código → texto), não do texto cru da API.
- Mensagem de validação de formulário ligada ao campo (`aria-describedby`, `aria-invalid`).
- Fetch manual, quando o repositório não tem biblioteca de dados: `AbortController` no cleanup do efeito e proteção contra resposta fora de ordem.

Proibições:
- `catch` que engole o erro e deixa o componente em estado de carregamento ou de sucesso falso.
- `alert`/`console.error` como único tratamento de erro visível.
- Efeito assíncrono sem cleanup quando há assinatura, timer ou requisição.

## Testes

Defaults:
- Testing Library com consultas por papel, rótulo e texto (`getByRole`, `getByLabelText`); `data-testid` só quando não há alternativa acessível.
- `userEvent` em vez de `fireEvent`; assíncrono com `findBy*`/`waitFor`, nunca espera arbitrária.
- Testa comportamento observável pelo usuário: o que aparece, o que acontece ao interagir. Não testa estado interno, nome de classe CSS nem chamadas de método do componente.
- Rede simulada na fronteira (MSW **se presente**; senão mock do cliente de API do repositório), não `fetch` global por teste.
- Hook customizado testado via `renderHook` ou pelo componente que o usa.
- Snapshot só para markup estático e pequeno.
- Jornada crítica (`VER-*` do tipo `e2e`): Playwright, com screenshot de evidência por estado verificado, conforme `playwright-e2e.md`. Se o repositório já adota outra ferramenta, siga-a com a mesma política de evidência.
- Nome do teste descreve comportamento e cita o `VER-###`.

Proibições:
- Shallow render ou acesso a `instance()`/estado interno.
- `.only`/`.skip` versionado; `waitFor` com timeout inflado para mascarar condição de corrida.
- Teste que depende de ordem ou de estado deixado por outro teste.

## Segurança

- `dangerouslySetInnerHTML` só com conteúdo sanitizado pela biblioteca que o repositório já usa; markdown sanitizado antes de renderizar.
- `href`/`src` construídos com dado do usuário validam o esquema (`http`/`https`); `javascript:` é bloqueado.
- Variáveis `VITE_*`, `NEXT_PUBLIC_*`, `REACT_APP_*` são públicas: nenhum segredo nelas.
- Token de sessão segue a estratégia do repositório (cookie `httpOnly` ou armazenamento existente); não introduzir `localStorage` para credencial onde não existe.
- Dado sensível não vai para log do console nem para ferramenta de analytics.
- Nenhuma dependência nova sem autorização; lockfile intocado fora do escopo.

## Desempenho

- `useMemo`/`useCallback` só quando há custo medido ou um filho memoizado/dependência de efeito precisa de referência estável. Memoização por reflexo é ruído.
- Valor de `Context.Provider` memoizado quando é objeto ou função.
- Lista longa (centenas de itens visíveis) usa virtualização **se a biblioteca já existir**; senão, paginação.
- Divisão de código por rota com `lazy`/`Suspense` ou o mecanismo do framework.
- Imagem com dimensões definidas e componente otimizado do framework quando houver.
- Nenhuma cascata de `setState` em efeitos encadeados; se dois estados mudam juntos, são um só ou derivados.
- Objeto/array literal criado no render e passado como prop ou dependência só quando não causa re-render de filho memoizado.

## Armadilhas frequentes

- Closure obsoleta em handler/efeito que lê estado antigo: dependência faltando ou forma funcional de `setState` ausente.
- Buscar dados em `useEffect` sem abortar: resposta antiga sobrescreve a nova.
- `StrictMode` executa efeitos duas vezes em desenvolvimento: efeito sem cleanup idempotente se revela aqui, não é bug do React.
- `async` direto na função do `useEffect` (retorna promise, não cleanup).
- Prop com valor padrão de objeto (`items = []`) gera referência nova a cada render.
- Esquecer `key` em `Fragment` dentro de `map`.
- Armazenar em estado o que é derivável de props ou de outro estado.
- Em Next.js: hook ou `window` em componente servidor; `"use client"` colocado alto demais na árvore.

## Checagens para review

- [ ] Nenhum componente de classe novo; hooks sempre no topo e incondicionais.
- [ ] `useEffect` novo sincroniza algo externo; nenhum efeito que só copia prop/estado para estado.
- [ ] `exhaustive-deps` respeitado ou desabilitado com justificativa escrita.
- [ ] Estado de servidor na biblioteca de dados do repositório; fetch manual, se existir, tem abort e proteção de ordem.
- [ ] Estados de carregamento, erro e vazio tratados em toda visão assíncrona nova.
- [ ] `key` estável; nenhuma mutação de estado; nenhum formulário misto.
- [ ] Elementos semânticos, `label` ligado, `alt` presente, operação por teclado possível.
- [ ] `dangerouslySetInnerHTML` ausente ou com sanitização existente; URLs de usuário validadas; nenhum segredo em variável pública.
- [ ] Memoização só com motivo; valor de contexto memoizado quando é objeto/função.
- [ ] Cada `VER-*` da task existe com Testing Library por papel/rótulo, `userEvent`, rede simulada na fronteira e sem `data-testid` evitável.
- [ ] Nenhum `.only`, `.skip` novo, snapshot de lógica ou asserção enfraquecida.
