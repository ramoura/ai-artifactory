# Critérios de engenharia — aplicar por pertinência

Este documento **não impõe números universais**. Requisitos mais específicos da história, contratos, SLOs e normas existentes do projeto prevalecem. Registre `aplicável`, `não aplicável` ou `não verificado` e a justificativa, sem preencher o plano com checklist genérico.

## 1. Evidências do código

- Confirmar stack, versões, módulos, pacotes, configuração, build e pipeline pelos arquivos presentes.
- Identificar e citar exemplos reais de implementação, testes, logs, métricas e tratamento de erro.
- Reutilizar padrões existentes, a menos que sejam incompatíveis com o requisito; justificar divergência.
- Não supor recursos de dependências apenas pelo nome de frameworks.
- Para cada componente ou dependência nova proposta, registrar escopo/termos da busca, equivalentes encontrados e decisão de reutilizar, adaptar ou criar. Ausência de acesso não comprova ausência de equivalente.
- Nas camadas afetadas, observar diretórios, nomenclatura, assinaturas, erros, logging, configuração, testes e dependências conforme pertinência. Mudanças transversais precisam de evidências de cada camada, não de um único exemplo isolado.

## 2. Comportamento e contratos

- Definir entrada, saída, estados, validação, erros, idempotência e compatibilidade com consumidores existentes.
- Para bugs: evidência, gatilho, comportamento esperado, hipótese de causa e teste de regressão.
- Para mensageria: semântica de entrega, retries, DLQ, reprocessamento, duplicatas e ordenação quando aplicáveis.
- Para dados: esquema, migração, unicidade, consistência, reversibilidade e retenção quando aplicáveis.

## 3. Segurança e conformidade

- Revisar autenticação, autorização, validação de dados, injeção, exposição indevida e superfícies afetadas.
- Evitar segredos em código, logs e exemplos. Proteger PII e dados confidenciais.
- Prever rastreabilidade e trilhas de auditoria nos fluxos que as exigem, sem logar conteúdo sensível.
- Considerar limites de requisições, abuso e isolamento entre usuários/tenants se pertinente.
- Mudanças sensíveis exigem revisão humana e respeito às políticas corporativas.

## 4. Desempenho e resiliência

**Classificação:**
- `hot`: caminho de alta frequência/latência crítica confirmado por arquitetura ou dados.
- `cold`: configuração, administração, processamento eventual ou baixa frequência.
- `não aplicável`: alteração sem impacto de runtime.

Se `hot`:
- Registrar RPS e latências atuais, p95/p99, SLO, capacidade e error budget **somente se conhecidos**.
- Descrever impacto de I/O, CPU, memória, concorrência, conexões, timeout, retry, cache, circuit breaker e backpressure conforme código.
- Em Spring WebFlux, atenção a operações bloqueantes no event loop, composição Reactor, timeouts, cancelamento, WebClient, pool de conexões e políticas de erro. Não transformar tudo em `subscribeOn` sem medir e entender o scheduler.
- Justificar estratégias de teste de carga e critério quantitativo com fonte real.
- Se faltarem valores relevantes, registrar a lacuna; não inventar baselines como 5000 RPS ou metas de 92% de cobertura.

### Impacto quantitativo e prontidão

Quando a mudança alterar carga, fan-out, concorrência ou consumo de recursos:

- Registrar valores atuais e projetados, unidade, fonte e escopo (por instância ou agregado); distinguir medição de estimativa e explicitar suas premissas.
- Para dependentes afetados, estimar `chamadas por request × RPS` antes/depois, incluindo tentativas adicionais de retry quando pertinentes. Comparar o pico projetado com limites/capacidade conhecidos; considerar outras cargas que compartilhem o dependente.
- Avaliar conexões/concorrência, payload e limites de memória conforme o recurso afetado. Para processamento eventual, usar volume por lote, duração e janela de execução em vez de forçar RPS.
- Explicitar comportamento sob lentidão, indisponibilidade e saturação e reusar timeout, pool, limites e mecanismos de resiliência existentes. Propor verificação com cenário e critério de aceitação fundamentados.

**Bloqueia `ready`:** falta um dado necessário para escolher ou dimensionar a solução, demonstrar atendimento a requisito/SLO declarado, ou avaliar aumento de carga contra um limite relevante. Não inventar capacidade para encerrar a análise.

**Pode permanecer não bloqueante:** a evidência mostra que a mudança não aumenta a demanda nem altera limites relevantes, ou a lacuna não interfere na decisão técnica nem no aceite. Registrar justificativa e como/quando confirmar; escrever apenas `a definir` não basta. Se a descoberta puder invalidar o design, tratar como bloqueio.

Caminhos `cold` podem exigir melhorias fundamentadas em memória, custo ou janela de execução; a frequência isolada não justifica nem proíbe otimização.

## 5. Qualidade e verificações

- Requisito funcional: associar a comportamento observável e cenário de teste.
- Unitário: regras de negócio, entradas inválidas, estados e casos de borda.
- Integração: banco, filas, parceiros, cache, serialização, configurações e falhas.
- Contrato: formatos, códigos, compatibilidade entre serviços, eventos e APIs.
- E2E: fluxos críticos do usuário se viável no ambiente.
- Carga/segurança: apenas conforme risco e critérios mensuráveis.
- Metas de cobertura: usar política existente do projeto; não impor porcentagem arbitrária.
- Planos não devem afirmar que testes executaram. Identifique testes **propostos**, não resultados.

## 6. Observabilidade e operação

Se aplicável, descrever como verificar sucesso/falha por logs, métricas e traces sem expor dados. Explicitar estratégia de rollout/feature flag, monitoramento, rollback, runbook e alertas quando o risco exigir.

## 7. Escopo e simplicidade

- Registrar não objetivos e compatibilidade preservada.
- Não criar múltiplas camadas, filas, caches ou abstrações sem problema concreto.
- Planejar a menor mudança verificável, sem perder os cenários relevantes.
- Separar dependências reais de uma ordem convencional; evitar falsa serialização.
- Identificar código, configuração, flags, dependências e documentação obsoletos e planejar remoção com compatibilidade preservada; declarar quando não há remoções.
- Separar migrações e melhorias que não sejam necessárias para atender à demanda.
