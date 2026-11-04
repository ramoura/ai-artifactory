# Testes de interface com Playwright e evidência visual

Vale para qualquer front-end da pipeline (React e outros). Lida por `/planning` (ao escrever os `VER-*` do tipo `e2e`), `/implementation` (ao escrever e rodar), `/task-review` e `/integration-validation` (ao conferir a evidência). Precedência e semântica de **default** / **proibição**: `README.md` deste diretório.

## Quando usar

- Todo `VER-*` com `type: e2e` que passa por interface web usa Playwright, salvo decisão documentada do repositório por outra ferramenta (Cypress, por exemplo). Nesse caso, siga o repositório e aplique a mesma política de evidência com a ferramenta dele.
- Playwright precisa estar no repositório (`@playwright/test` em `devDependencies` e `playwright.config.*`). Se não estiver, a `implementation` **não instala**: marca o `VER-*` como `not_run` com motivo "Playwright ausente" e o orquestrador decide. A `planning` deve apontar essa ausência como blocker ou premissa quando o `tests.md` exigir E2E.
- Jornada de usuário (`REQ-*`) crítica: pelo menos um E2E do início ao fim. Comportamento de componente isolado continua em Testing Library; E2E não substitui teste unitário.

## Dois tipos de captura, não confundir

| Tipo | Para quê | Default |
| --- | --- | --- |
| **Screenshot de evidência** (`page.screenshot`) | Registrar a tela em cada estado verificado, para humano e reviewer olharem | **Sempre**, em todo `VER-*` e2e |
| **Comparação visual** (`expect(page).toHaveScreenshot()`) | Regressão de layout contra baseline versionada | Só se o repositório já mantém baselines; não introduzir por conta própria (frágil entre SO, fonte e resolução) |

Screenshot de evidência **não é asserção**. O teste continua afirmando o comportamento por `expect` em locators; a imagem documenta o que o `expect` validou.

## Política de screenshots (defaults)

- **Uma captura por estado verificado**, logo depois do `expect` que o confirma: tela inicial, após ação principal, estado de erro, estado vazio. Nunca uma só no fim.
- **Nome:** `VER-###-NN-<estado>.png`, por exemplo `VER-020-01-lista-vazia.png`, `VER-020-02-despesa-criada.png`. `NN` é a ordem dentro do caso.
- **Destino:** `test-results/evidence/` (configurado em `playwright.config` como `outputDir` ou via helper). A `implementation` copia as imagens do caso para `tasks/<slug>/results/TASK-###/screenshots/` e lista cada uma em `result.yaml` (`verification[].artifacts`). A `integration-validation` grava as suas em `tasks/<slug>/validation/screenshots/run-NN/`.
- **Tamanho:** viewport fixo (`1280x720` se o repositório não define) e `fullPage: true` quando a tela rola. Não capturar vídeo por padrão; `trace: 'on-first-retry'` e `screenshot: 'only-on-failure'` na config cobrem o diagnóstico de falha.
- **Dados:** massa de teste sintética. A captura entra no artefato da task; nenhuma PII, token ou dado real pode aparecer na tela.
- **Determinismo:** antes de capturar, aguarde o estado por locator (`await expect(locator).toBeVisible()`), nunca por tempo. Animações desligadas na config (`reducedMotion: 'reduce'`) quando a captura serve de comparação.

Helper sugerido (adaptar ao repositório; não duplicar se já existir equivalente):

```ts
// e2e/support/evidence.ts
import { Page, TestInfo } from '@playwright/test';
export async function evidence(page: Page, info: TestInfo, ver: string, step: number, state: string) {
  const file = `${ver}-${String(step).padStart(2, '0')}-${state}.png`;
  const path = info.outputPath('evidence', file);
  await page.screenshot({ path, fullPage: true });
  await info.attach(file, { path, contentType: 'image/png' });
  return path;
}
```

## Escrita dos testes (defaults)

- Locators por papel e nome acessível (`getByRole`, `getByLabel`, `getByText`); `getByTestId` só quando o repositório já o adota para aquele elemento.
- Um caso por `VER-*`; o título do teste começa com o ID: `test('VER-020 cria despesa e vê na lista', ...)`.
- Page objects ou fixtures conforme o repositório; não criar camada nova para um único teste.
- Autenticação por `storageState` gerado em setup, não por login repetido em cada teste.
- Dados criados pelo próprio teste (API de setup, seed ou fixture) e limpos ao final; nenhum teste depende de outro.
- Rede externa ao sistema sob teste simulada com `page.route` ou servidor fake do repositório.
- Execução headless; `--headed` e `--debug` só localmente.
- Comando: o script do repositório (`npm run test:e2e` ou equivalente). Registrar o comando real em `result.yaml`.

## Proibições

- `page.waitForTimeout` ou `sleep` para sincronizar.
- Seletor por CSS frágil (`div > div:nth-child(3)`) ou por texto de classe quando existe papel acessível.
- `test.only`/`test.skip` versionado; `expect.soft` para mascarar falha de requisito.
- Screenshot como única verificação do caso (teste sem `expect` em locator).
- Instalar `@playwright/test` ou navegadores (`npx playwright install`) sem autorização.
- Capturar tela com dado real de usuário.

## Checagens para review

- [ ] Cada `VER-*` e2e da task existe como `test('VER-### ...')`, com `expect` em locator para o comportamento de `tests.md`.
- [ ] Há screenshot de evidência para cada estado verificado, nomeada `VER-###-NN-<estado>.png`, listada em `result.yaml` e presente em `results/TASK-###/screenshots/`.
- [ ] O reviewer **abriu** as imagens e confirmou que mostram o estado descrito (tela, dado, mensagem). Imagem que não corresponde ao `expect` vira finding; imagem ausente torna o `VER-*` `not_verified`.
- [ ] Nenhum `waitForTimeout`, `test.only`, seletor frágil ou PII em captura.
- [ ] Comparação visual só se o repositório já tinha baselines; nenhuma baseline nova sem autorização.
- [ ] Comando de execução real registrado; `not_run` com motivo se Playwright ou navegador estava indisponível.
