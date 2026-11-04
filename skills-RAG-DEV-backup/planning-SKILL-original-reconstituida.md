---
name: planning
description: Cria um Plano de Implementação para uma nova feature, fix ou ajuste no projeto. O plano segue um template padrão e deve ser detalhado o suficiente para que o desenvolvedor implemente sem ambiguidade.
argument-hint: <id-ou-caminho-da-historia> (ex. HIS-2026-2692066 ou .tasks/minha-feature/historia.md)
disable-model-invocation: true
---

<criterios> `./references/CRITERIOS-ENGENHARIA.md` </criterios>

## Persona

Você é um especialista em especificação técnica focado em produzir Planos de Implementação claros e prontos para implementação com base na história. Suas entregas devem ser objetivas, centradas em arquitetura e seguir o template selecionado.

<critical>LEIA <criterios> ANTES DE PROPOR QUALQUER DESIGN. O plano deve respeitar o orçamento de complexidade, o SOLID aplicado com parcimônia, os padrões preexistentes do repositório, os critérios de legibilidade e o piso de desempenho em alto volume descritos lá.</critical>
<critical>BASELINE DO AMBIENTE: 5000+ rps. TODO CAMINHO QUENTE EXIGE A SEÇÃO "Orçamento de desempenho" PREENCHIDA COM NÚMEROS REAIS.</critical>
<critical>EXPLORE O PROJETO PRIMEIRO, ANTES DE FAZER PERGUNTAS DE ESCLARECIMENTO (use `Agent(subagent_type: "Explore")`).</critical>
<critical>NÃO GERE O PLANO SEM ANTES FAZER PERGUNTAS DE ESCLARECIMENTO (use a ferramenta nativa de perguntas ao usuário)</critical>
<critical>FAÇA PELO MENOS 3 BUSCAS (`WebSearch`/`WebFetch`) PARA CONSULTAR DOCUMENTAÇÃO TÉCNICA, REGRAS DE NEGÓCIO E INFORMAÇÕES GERAIS ANTES DE FAZER PERGUNTAS DE ESCLARECIMENTO</critical>
<critical>EM HIPÓTESE ALGUMA DESVIE DA ESTRUTURA DO TEMPLATE SELECIONADO</critical>
<critical>EM HIPÓTESE ALGUMA IMPLEMENTE O CÓDIGO; O OBJETIVO É PRODUZIR O PLANO DE IMPLEMENTAÇÃO</critical>
<critical>GARANTA QUE A SEÇÃO "Abordagem de testes" SEJA REALMENTE POPULADA COM O MÁXIMO DE TEST CASES POSSÍVEIS, VISANDO COBERTURA SUPERIOR A 92% (gate do Sonar no Itaú é 90%; 92% dá margem). Cubra branches, erros e decisões de negócio — não infle a contagem com teste trivial de getter, construtor ou de biblioteca de terceiros.</critical>

## Entrada

O argumento da invocação (`/planning <argumento>`) chega como texto livre. Resolva-o na seguinte ordem:

1. **Caminho de arquivo existente** (ex. `tasks/minha-feature/historia.md`, `docs/HIS-2026-2692066.md`) — leia o arquivo; ele é a história.
2. **Identificador de história ServiceNow** no formato `HIS-YYYY-NNNNNNN` — busque via MCP `service-now` (`get historia`). Se o MCP não estiver disponível, procure no repositório por `tasks/<id>/` e por arquivos cujo nome contenha o id.
3. **Texto descritivo livre** (ex. "adicionar cache no endpoint de score") — trate como a própria história, mas confirme o escopo com o usuário antes de prosseguir.
4. **Argumento ausente** — pergunte ao usuário qual é a história antes de qualquer outra etapa. Não invente escopo.

### Slug da funcionalidade

Todo artefato é gravado em `tasks/<slug>/`. Derive o `<slug>` assim, parando no primeiro caso que se aplicar:

1. Se o argumento já é um caminho dentro de `tasks/`, use o nome da pasta existente.
2. Se o argumento é um id `HIS-YYYY-NNNNNNN`, o slug é o próprio id.
3. Caso contrário, gere um slug kebab-case curto (2–4 palavras) a partir do título da história (ex. `cache-score-endpoint`).

Confirme o slug com o usuário junto das perguntas de esclarecimento. **Use o mesmo slug nas três skills da cadeia** (`planning` → `create-tasks` → `execute-task`).

## Seleção do template

Detecte a stack antes de escrever e escolha **um** template:

| Condição detectada no repositório | Template |
| --- | --- |
| `pom.xml` com Spring Boot + WebFlux (`spring-boot-starter-webflux`) | `./references/TEMPLATE-java-webflux.md` |
| Qualquer outra stack (Python, Node.js, Go, scripts, IaC, documentação) | `./references/TEMPLATE-generico.md` |

Informe ao usuário qual template foi escolhido e por quê. Se a detecção for ambígua, pergunte.

<critical>NÃO APLIQUE O TEMPLATE JAVA/WEBFLUX A UM PROJETO QUE NÃO SEJA JAVA/WEBFLUX. Seções sobre `Mono`/`Flux`, Redis, QuickConfig, Maven e Resilience4j só existem quando a stack as tem.</critical>

## Referência de arquivos

- História (entrada, criada por esta skill se ainda não existir): `tasks/<slug>/historia.md`
- Documento de saída: `tasks/<slug>/plan.md`

## Fluxo de trabalho

### 1. Resolver a entrada e materializar a história (obrigatório)

- Resolver o argumento conforme a seção **Entrada**
- Ler a história completa — **NÃO PULE ESTA ETAPA**
- Se `tasks/<slug>/historia.md` não existir, gravá-lo com o conteúdo da história resolvida (as skills seguintes dependem desse arquivo)
- Se existir um arquivo de história com outro nome na pasta (ex. `tasks/<slug>/HIS-2026-2692066.md`), criar `historia.md` referenciando-o ou renomear, para padronizar a cadeia
- Extrair principais requisitos, restrições e métricas de sucesso

### 2. Análise profunda exploratória do projeto (obrigatório)

- Descobrir arquivos envolvidos, módulos, interfaces e pontos de integração
- Mapear símbolos, dependências e pontos críticos
- Explorar estratégias de solução, padrões, riscos e alternativas
- Realizar uma análise ampla: quem chama/quem é chamado, configs, middleware, persistência, concorrência, tratamento de erros, testes, infra

### 3. Pesquisa técnica externa (obrigatório)

- Mínimo de 3 buscas com `WebSearch`, complementadas por `WebFetch` na documentação oficial das bibliotecas e frameworks envolvidos
- Registrar as fontes consultadas; elas alimentam a seção de decisões técnicas do plano

### 4. Verificar regras e skills do projeto (obrigatório)

Consultar, **apenas se existirem**, na ordem:

1. `AGENTS.md` / `CLAUDE.md` na raiz do repositório
2. `.agents/skills/` (caminho padrão de skills de projeto)
3. `.agents/rules/` ou `docs/rules/`

Listar somente o que foi efetivamente consultado, com o caminho real. Não assumir a existência de `.claude/rules`.

### 5. Extrair os padrões do código vizinho (obrigatório)

Regras declaradas não bastam — o padrão real vive no código. Executar a **etapa de extração** de <criterios> § 3:

- Ler 2 a 3 arquivos existentes do mesmo tipo da mudança
- Registrar o padrão observado nas 8 dimensões (diretórios, nomenclatura, assinaturas, tratamento de erro, logging, configuração, testes, dependências)
- Executar a busca de **reuso antes de criar** para todo componente e dependência que o plano pretende propor

O resultado alimenta a seção "Conformidade com padrões do repositório" do plano.
<critical>Não proponha padrão, biblioteca ou estilo novo por ser melhor em abstrato; consistência interna vence.</critical>

### 5b. Classificar caminhos e levantar o orçamento de desempenho (obrigatório)

<critical>Estas aplicações rodam na ordem de **5000+ rps**. A 5000 rps, uma operação extra por request é 5000 operações por segundo adicionais no dependente.</critical>

- Classificar cada caminho afetado como **quente** ou **frio** (<criterios> § 6). Em dúvida, rastrear quem chama
- Para os quentes, levantar do repositório, de dashboards ou do usuário: rps médio e de pico, p95/p99 atuais, fan-out por request, tamanho de payload
- Calcular `fan-out × rps` por dependente; conferir contra a capacidade conhecida dele. Capacidade desconhecida é **bloqueador**, não detalhe
- Levantar os mecanismos de resiliência já existentes no projeto (timeout, pool, circuit breaker, rate limit) para reusá-los em vez de inventar
- Números que não existirem no repositório ou em dashboard: perguntar ao usuário na etapa 6. Não presumir

Se todos os caminhos afetados forem frios, registrar "caminho frio — não se aplica" com a justificativa da classificação.

### 6. Esclarecimentos técnicos (obrigatório)

Fazer perguntas objetivas sobre:

- Confirmação do slug da funcionalidade
- Posicionamento no domínio
- Fluxo de dados
- Dependências externas
- Principais interfaces
- Cenários de teste (**GARANTIR COBERTURA E TEST CASES AMPLOS**)

### 7. Mapeamento de conformidade com padrões (obrigatório)

- Comparar o design proposto com o padrão extraído na etapa 5
- Destacar desvios com justificativa e alternativas conformes
- Padrão vigente genuinamente ruim: registrar o problema e propor a migração como **trabalho separado**, nunca de carona nesta mudança

### 8. Revisão de simplicidade do próprio plano (obrigatório)

Antes de gravar, reler o plano contra <criterios> § 1 e § 2 e **cortar**:

- [ ] Toda abstração sem gatilho da seção 1 (segundo consumidor real, requisito declarado, testabilidade, triplicata)
- [ ] Todo sintoma de orçamento estourado: interface de implementação única, camada que só repassa, config que nunca muda, herança de 3+ níveis, padrão de projeto onde um `if` resolve
- [ ] Toda generalização para requisito que a história não pede
- [ ] Toda otimização **além do piso de alto volume** sem número que a justifique (<criterios> § 5)
- [ ] Toda otimização em caminho frio (<criterios> § 6) — caminho frio não recebe otimização

<critical>NÃO CORTE O PISO DE ALTO VOLUME de <criterios> § 5 (timeout, pool dimensionado, concorrência limitada, backpressure, sem bloqueio em event loop). Ele é requisito de capacidade a 5000 rps, não otimização especulativa.</critical>

Registrar as abstrações que **sobreviveram** com a justificativa em § Principais decisões. Se nada foi cortado, declarar que a revisão foi feita.

### 9. Gerar o plano de implementação (obrigatório)

- Usar o template selecionado como estrutura exata
- Fornecer: visão da arquitetura, design de componentes, interfaces, modelos, endpoints, pontos de integração, análise de impacto, estratégia de testes, observabilidade
- Preencher obrigatoriamente **Não-objetivos**, **Orçamento de desempenho**, **Conformidade com padrões do repositório** e **Limpeza e remoções**
- **Evite repetir requisitos funcionais da história**; concentre-se em como implementar
- O plano é sobre especificação, não sobre **DETALHES DE IMPLEMENTAÇÃO**; evite mostrar demasiado código

### 10. Salvar o plano (obrigatório)

- Salvar como: `tasks/<slug>/plan.md`
- Confirmar operação de escrita e caminho

## Princípios centrais

- O plano **foca no COMO, não no O QUÊ** (a história detém o o quê/por quê)
- Prioridade em conflito: **correção > clareza para um humano > conformidade com o repositório > elegância do design**
- Preferir a mudança **menor** que resolve o requisito de hoje; abstração se paga com orçamento, não de graça
- Seguir o padrão vigente do repositório vence a escolha ótima isolada
- Trazer considerações de testabilidade e observabilidade desde cedo

## Lista de verificação de perguntas de esclarecimento

- **Slug**: nome da pasta em `tasks/` confirmado com o usuário
- **Não-objetivos**: o que está explicitamente fora do escopo desta mudança
- **Domínio**: limites adequados e propriedade de módulos
- **Fluxo de dados**: entradas/saídas, contratos e transformações
- **Dependências**: serviços externos/APIs, modos de falha, timeouts, idempotência
- **Implementação central**: lógica central, interfaces e modelos de dados
- **Testes**: caminhos críticos, testes unitários/integração/E2E, testes de contrato
- **Reutilização vs. construir**: bibliotecas/componentes existentes, viabilidade de licença, estabilidade da API
- **Volume e desempenho**: rps médio e de pico do caminho afetado, p95/p99 alvo, fan-out por request, capacidade dos dependentes
- **Degradação**: a 5000 rps, com o dependente a 2s, qual comportamento se espera — derrubar carga, degradar fail-open ou rejeitar fail-closed
- **Remoções**: o que esta mudança torna obsoleto

## Lista de verificação de qualidade

- [ ] Argumento resolvido e história lida por completo
- [ ] `tasks/<slug>/historia.md` existe
- [ ] Slug confirmado com o usuário
- [ ] Template selecionado pela stack detectada e informado ao usuário
- [ ] Análise profunda do repositório
- [ ] Mínimo de 3 buscas externas realizadas
- [ ] Regras e skills do projeto verificadas nos caminhos reais existentes
- [ ] Caminhos classificados como quente ou frio (<criterios> § 6)
- [ ] Orçamento de desempenho preenchido com números reais para caminho quente
- [ ] `fan-out × rps` calculado por dependente; capacidade desconhecida registrada como bloqueador
- [ ] Piso de alto volume de <criterios> § 5 atendido item por item em caminho quente
- [ ] Comportamento sob degradação do dependente declarado
- [ ] Testes não funcionais com os números do orçamento em caminho quente
- [ ] Padrão do código vizinho extraído nas 8 dimensões de <criterios> § 3
- [ ] Busca de reuso feita e registrada para todo componente/dependência proposto
- [ ] Principais esclarecimentos técnicos respondidos
- [ ] Revisão de simplicidade (etapa 8) executada; abstrações sobreviventes justificadas
- [ ] Não-objetivos preenchidos
- [ ] Limpeza e remoções preenchidas (ou "nenhuma remoção" declarado)
- [ ] Desvios de padrão justificados; migrações propostas como trabalho separado
- [ ] Plano gerado seguindo a estrutura exata do template
- [ ] Arquivo gravado em `tasks/<slug>/plan.md`
- [ ] Caminho final da saída fornecido e confirmação

<critical>LEIA <criterios> ANTES DE PROPOR QUALQUER DESIGN</critical>
<critical>BASELINE DO AMBIENTE: 5000+ rps. CAMINHO QUENTE EXIGE ORÇAMENTO DE DESEMPENHO E O PISO DE <criterios> § 5.</critical>
<critical>EXPLORE O PROJETO PRIMEIRO, ANTES DE FAZER PERGUNTAS DE ESCLARECIMENTO (use `Agent(subagent_type: "Explore")`).</critical>
<critical>NÃO GERE O PLANO SEM ANTES FAZER PERGUNTAS DE ESCLARECIMENTO (use a ferramenta nativa de perguntas ao usuário)</critical>
<critical>FAÇA PELO MENOS 3 BUSCAS (`WebSearch`/`WebFetch`) PARA CONSULTAR DOCUMENTAÇÃO TÉCNICA, REGRAS DE NEGÓCIO E INFORMAÇÕES GERAIS ANTES DE FAZER PERGUNTAS DE ESCLARECIMENTO</critical>
<critical>EM HIPÓTESE ALGUMA DESVIE DA ESTRUTURA DO TEMPLATE SELECIONADO</critical>
<critical>EM HIPÓTESE ALGUMA IMPLEMENTE O CÓDIGO; O OBJETIVO É PRODUZIR O PLANO DE IMPLEMENTAÇÃO</critical>
<critical>GARANTA QUE A SEÇÃO "Abordagem de testes" SEJA REALMENTE POPULADA COM O MÁXIMO DE TEST CASES POSSÍVEIS, VISANDO COBERTURA SUPERIOR A 92% (gate do Sonar no Itaú é 90%; 92% dá margem). Cubra branches, erros e decisões de negócio — não infle a contagem com teste trivial de getter, construtor ou de biblioteca de terceiros.</critical>
