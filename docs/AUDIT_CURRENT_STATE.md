# Auditoria do Estado Atual

Data da auditoria: 2026-06-26

Escopo: leitura estrutural do projeto em `D:\TradingAgents-main`, sem alteracao de codigo.

## Resumo executivo

O projeto aberto atualmente e uma base `TradingAgents` em Python, versao declarada `0.2.4`, com um framework multiagente para analise de ativos financeiros usando LangGraph, LangChain, provedores LLM multiplos, ferramentas de dados de mercado, memoria persistente e CLI interativa.

Ele ja tem uma fundacao tecnica real para agentes de analise/trading, mas ainda nao e um "Hermes Agent" completo no sentido descrito no transcript. Nao ha API local, MCP server, camada blockchain/wallet, engine de anomalias on-chain, desktop terminal, SaaS, `AGENTS.md`, `CLAUDE.md` ou documentacao arquitetural propria.

Recomendacao principal: preservar a arquitetura atual de grafo, clientes LLM, dataflows, memoria e testes, fazer um checkpoint Git antes de qualquer mudanca, corrigir inconsistencias de ambiente/documentacao, e so depois desenhar a camada Hermes/API/MCP por fora do core existente.

## Estado do repositorio

- Pasta raiz analisada: `D:\TradingAgents-main`
- Git: nao ha `.git` nesta pasta no momento da auditoria.
- `docs/`: nao existia antes desta auditoria.
- Estrutura de monorepo Hermes (`apps/`, `packages/`, `tools/`, `.mcp.json`, `.codex/`): ausente.
- Arquivos de orientacao para agentes (`AGENTS.md`, `CLAUDE.md`): ausentes.

## Estrutura encontrada

```text
D:\TradingAgents-main
|-- assets/
|-- cli/
|-- reports/
|-- scripts/
|-- tests/
|-- tradingagents/
|   |-- agents/
|   |   |-- analysts/
|   |   |-- managers/
|   |   |-- researchers/
|   |   |-- risk_mgmt/
|   |   |-- trader/
|   |   `-- utils/
|   |-- dataflows/
|   |-- graph/
|   `-- llm_clients/
|-- .env.example
|-- .env.enterprise.example
|-- Dockerfile
|-- docker-compose.yml
|-- main.py
|-- pyproject.toml
|-- README.md
|-- requirements.txt
|-- test.py
`-- uv.lock
```

## Modulos existentes

### Core de orquestracao

Arquivos principais:

- `tradingagents/graph/trading_graph.py`
- `tradingagents/graph/setup.py`
- `tradingagents/graph/propagation.py`
- `tradingagents/graph/conditional_logic.py`
- `tradingagents/graph/checkpointer.py`
- `tradingagents/graph/reflection.py`
- `tradingagents/graph/signal_processing.py`

O core usa LangGraph para encadear:

1. analistas selecionaveis: market, social, news, fundamentals;
2. pesquisadores bull/bear;
3. research manager;
4. trader;
5. analistas de risco aggressive/conservative/neutral;
6. portfolio manager.

Pontos fortes:

- Separacao razoavel entre setup do grafo, propagacao, logica condicional, checkpoint e processamento de sinal.
- Checkpoint por ticker usando SQLite via `langgraph-checkpoint-sqlite`.
- `TradingAgentsGraph` e uma boa fronteira para uma futura API local.

Pontos de atencao:

- `TradingAgentsGraph.__init__` cria diretorios em `~/.tradingagents` ao instanciar o grafo.
- O metodo `_log_state` grava JSON em disco ao final de cada execucao.
- O grafo atual e orientado a tickers e dados financeiros tradicionais, nao a blockchain/on-chain.

### Agentes

Arquivos principais:

- `tradingagents/agents/analysts/*.py`
- `tradingagents/agents/researchers/*.py`
- `tradingagents/agents/managers/*.py`
- `tradingagents/agents/risk_mgmt/*.py`
- `tradingagents/agents/trader/trader.py`
- `tradingagents/agents/schemas.py`

Pontos fortes:

- Agentes de decisao principais usam schemas Pydantic e structured output.
- Research Manager, Trader e Portfolio Manager renderizam markdown padronizado.
- Ha fallback para free text quando structured output nao esta disponivel.

Pontos de atencao:

- Prompts estao bem acoplados ao dominio de trading em equities/empresas.
- A ideia de cripto/blockchain ainda nao aparece como papel oficial de agente.
- Ainda nao ha distincao formal entre modo estudo, desenvolvimento, producao e trade real.

### Clientes LLM

Arquivos principais:

- `tradingagents/llm_clients/base_client.py`
- `tradingagents/llm_clients/factory.py`
- `tradingagents/llm_clients/openai_client.py`
- `tradingagents/llm_clients/google_client.py`
- `tradingagents/llm_clients/anthropic_client.py`
- `tradingagents/llm_clients/azure_client.py`
- `tradingagents/llm_clients/model_catalog.py`
- `tradingagents/llm_clients/validators.py`

Provedores suportados:

- OpenAI
- Google
- Anthropic
- xAI
- DeepSeek
- Qwen
- GLM
- OpenRouter
- Ollama
- Azure OpenAI

Pontos fortes:

- Factory lazy (`create_llm_client`) evita carregar SDKs desnecessarios.
- Conteudo de resposta e normalizado para string.
- Ha validacao de modelos com warning, nao erro duro.
- Boa fronteira para plugar configuracao futura sem espalhar logica nos agentes.

Pontos de atencao:

- `tradingagents/llm_clients/TODO.md` esta desatualizado: diz que `validate_model()` nunca e chamado, mas os clientes ja chamam `warn_if_unknown_model()`.
- URLs default diferem em alguns lugares: `cli/utils.py` usa `https://dashscope.aliyuncs.com/compatible-mode/v1`, enquanto `openai_client.py` usa `https://dashscope-intl.aliyuncs.com/compatible-mode/v1` para Qwen.

### Dataflows e ferramentas

Arquivos principais:

- `tradingagents/dataflows/interface.py`
- `tradingagents/dataflows/config.py`
- `tradingagents/dataflows/y_finance.py`
- `tradingagents/dataflows/yfinance_news.py`
- `tradingagents/dataflows/alpha_vantage_*.py`
- `tradingagents/agents/utils/*_tools.py`

Vendors existentes:

- `yfinance`
- `alpha_vantage`

Categorias:

- `core_stock_apis`
- `technical_indicators`
- `fundamental_data`
- `news_data`

Pontos fortes:

- Roteamento por categoria e por ferramenta.
- Fallback para outro vendor em caso de rate limit do Alpha Vantage.
- Cuidado contra look-ahead bias em dados de mercado e indicadores.

Pontos de atencao:

- Nao ha engine blockchain, wallet scanner, exchange connector ou detector on-chain.
- `ALPHA_VANTAGE_API_KEY` e exigido apenas quando vendor Alpha Vantage e usado.
- `redis` esta em dependencias, mas nao apareceu como parte ativa do runtime analisado.

### Memoria e logs

Arquivos principais:

- `tradingagents/agents/utils/memory.py`
- `tradingagents/graph/reflection.py`
- `tradingagents/graph/trading_graph.py`

Comportamento:

- Decisoes sao gravadas em markdown em `~/.tradingagents/memory/trading_memory.md` por default.
- Entradas pendentes sao resolvidas em execucao futura do mesmo ticker.
- Resultado realizado e alpha vs SPY sao usados para reflexao posterior.

Pontos fortes:

- Memoria append-only com delimitador explicito.
- Escrita atomica em atualizacoes.
- Contexto passado e injetado no Portfolio Manager.

Pontos de atencao:

- Memoria e especifica para decisoes de trading/ticker.
- Ainda nao ha memoria operacional geral do Hermes, logs de estudo, auditoria de comandos naturais, ou memoria multi-dominio.

### CLI

Arquivos principais:

- `cli/main.py`
- `cli/utils.py`
- `cli/models.py`
- `cli/stats_handler.py`
- `cli/announcements.py`

Pontos fortes:

- CLI interativa com Typer, Rich e Questionary.
- Permite escolher ticker, data, idioma, analistas, profundidade, provedor LLM e modelos.
- Exibe progresso, chamadas de ferramentas, uso de tokens e relatorios.
- Suporta `--checkpoint` e `--clear-checkpoints`.

Pontos de atencao:

- `cli/main.py` e grande (mais de 1200 linhas) e mistura UI, orquestracao, persistencia de relatorios e streaming.
- Existe `ANALYST_ORDER` em `cli/utils.py` com tuplas e outro `ANALYST_ORDER` em `cli/main.py` com strings. Funciona pelo escopo atual, mas cria confusao e risco de manutencao.
- `cli/announcements.py` consulta `api.tauric.ai/v1/announcements`; para um Hermes local/privacy-first isso deveria ser opcional e documentado.

### Scripts, exemplos e artefatos soltos

Arquivos:

- `main.py`
- `test.py`
- `scripts/smoke_structured_output.py`

Observacoes:

- `scripts/smoke_structured_output.py` e util e focado.
- `main.py` e `test.py` parecem exemplos/laboratorio na raiz.
- `main.py` ainda menciona `reflect_and_remember` em comentario, embora o changelog e testes indiquem que isso foi removido.
- `reports/` existe na raiz, mas estava vazio no momento da listagem.

### Testes

Diretorio:

- `tests/`

Cobertura observada:

- Memoria e reflexao diferida.
- Structured output.
- Validacao de modelos.
- Google API key.
- Checkpoint/resume.
- Processamento de rating/sinal.
- Normalizacao de ticker.

Pontos fortes:

- Ha testes unitarios relevantes para os componentes mais sensiveis.
- Testes usam mocks para evitar credenciais reais.

Pontos de atencao:

- Nao rodei a suite para evitar criacao de caches/artefatos enquanto ha trabalho paralelo em andamento.
- Nao identifiquei testes end-to-end da CLI interativa.
- Nao ha testes de API/MCP porque esses modulos ainda nao existem.

## Dependencias e ambiente

Fonte principal:

- `pyproject.toml`

Observacoes:

- `requirements.txt` contem apenas `.`.
- `pyproject.toml` declara versao `0.2.4`.
- `uv.lock` declara o pacote local como `0.2.0`, indicando lock possivelmente desatualizado.
- `uv.lock` ainda contem `chainlit` e `rank-bm25`.
- `CHANGELOG.md` informa que `chainlit` foi removido por CVE e que BM25 antigo foi removido.

Risco:

- Ambiente instalado por `uv.lock` pode divergir do ambiente instalado por `pip install .`.
- Isso e especialmente relevante antes de qualquer trabalho com MCP/API, porque `uv.lock` tambem contem `mcp`, `uvicorn` e pacotes relacionados por dependencia antiga/transitiva.

## O que ja existe vs. visao Hermes

### Ja existe

- Core multiagente com LangGraph.
- Analistas, pesquisadores, trader, risk analysts e portfolio manager.
- Multi-provider LLM.
- Ferramentas de dados de mercado via yfinance/Alpha Vantage.
- Memoria de decisoes e reflexao diferida.
- Checkpoint/resume por ticker.
- CLI rica.
- Docker e docker-compose.
- Testes relevantes.

### Ainda nao existe

- API local Hermes.
- MCP server.
- `.mcp.json`.
- Configuracao Codex/Claude especifica do projeto.
- `AGENTS.md` e `CLAUDE.md`.
- Documentacao arquitetural propria (`ARCHITECTURE.md`, `ROADMAP.md`, `SECURITY.md`).
- Engine blockchain/on-chain.
- Wallet scanner.
- Detector de anomalias cripto.
- Backtest engine formal exposta como recurso do produto.
- Desktop terminal.
- SaaS.
- Controle formal de modos: estudo, desenvolvimento, producao, trade real.
- Camada de permissoes para operacoes sensiveis.

## Riscos principais

### Alto

1. Ausencia de versionamento Git nesta pasta.

Sem `.git`, trabalho paralelo entre Codex, Claude e usuario fica dificil de auditar, comparar e reverter com seguranca.

2. Divergencia entre `pyproject.toml` e `uv.lock`.

O lock referencia versao antiga do pacote e dependencias que o changelog diz terem sido removidas. Isso pode criar diferencas entre maquinas, comandos e agentes.

3. Misturar Hermes com TradingAgents sem decisao arquitetural.

A base atual e forte para trading tradicional, mas Hermes, pelo transcript, pretende incluir blockchain, cripto, anomalias, memoria operacional e SaaS. Implementar tudo diretamente dentro de `tradingagents/` pode gerar acoplamento e retrabalho.

4. Ausencia de limites formais para operacoes sensiveis.

Nao ha hoje camada explicita de permissao para impedir trade real, acesso a wallets, private keys ou execucao automatica. A base atual parece simulada/analitica, mas a fronteira de seguranca deve existir antes de qualquer integracao externa.

### Medio

1. CLI grande e concentrada.

`cli/main.py` mistura interface, streaming, status, persistencia e execucao. Para API/MCP, convem expor primeiro uma camada de servico reutilizavel, nao chamar a CLI por dentro.

2. Scripts soltos na raiz.

`main.py` e `test.py` podem confundir comandos oficiais com experimentos.

3. Documentacao tecnica insuficiente para colaboradores/agentes.

README e CHANGELOG sao bons, mas faltam docs internas de arquitetura, seguranca, roadmap e instrucoes para agentes.

4. Nomes duplicados/confusos.

`ANALYST_ORDER` existe em dois lugares com shapes diferentes. Hoje parece controlado pelo escopo, mas nao e ideal.

### Baixo

1. `reports/` vazio na raiz.

Nao e problema agora, mas relatorios gerados podem acabar versionados se o Git for iniciado com `git add .` sem revisao.

2. `requirements.txt` minimalista.

Funciona se a instalacao oficial for `pip install .`, mas pode confundir usuarios que esperam dependencias listadas ali.

## Recomendacoes

### Prioridade 0 - antes de qualquer mudanca de codigo

1. Criar checkpoint Git ou confirmar que o trabalho paralelo esta em outra copia/worktree.
2. Revisar `git add .` antes do commit para nao incluir relatorios, caches ou artefatos locais.
3. Definir se esta pasta continuara como `TradingAgents` ou sera promovida para base `Hermes`.

### Prioridade 1 - documentacao e governanca

1. Criar `docs/ARCHITECTURE_REVIEW.md`.
2. Criar `docs/NEXT_IMPLEMENTATION_PLAN.md`.
3. Criar `SECURITY.md` com regras para secrets, wallets, exchanges e trade real.
4. Criar `AGENTS.md` e `CLAUDE.md` com responsabilidades e limites.

### Prioridade 2 - higiene de projeto

1. Atualizar ou regenerar `uv.lock`.
2. Atualizar/remover `tradingagents/llm_clients/TODO.md`.
3. Mover `main.py` e `test.py` para `examples/` ou `scripts/`, ou documentar o papel deles.
4. Decidir se `reports/` deve ser ignorado ou mantido.
5. Alinhar URLs default de provedores entre CLI e clientes.

### Prioridade 3 - preparacao para Hermes sem MCP ainda

1. Definir uma interface de servico em torno de `TradingAgentsGraph`.
2. Separar o que e "core de analise" do que e CLI.
3. Definir um modelo de permissao para acoes sensiveis.
4. Criar conceitos de modo: estudo, desenvolvimento, producao, trade real.
5. Criar logs/auditoria de execucoes em formato estavel.

### Prioridade 4 - API local

Somente depois das prioridades anteriores:

1. Criar uma API local fina que chame o core sem depender da CLI.
2. Expor endpoints seguros como status, memoria, analise e relatorios.
3. Nao expor trade real, wallets ou secrets.

### Prioridade 5 - MCP

Somente depois de API local e seguranca:

1. Criar MCP server como ponte fina para a API.
2. Ferramentas iniciais recomendadas:
   - `hermes_get_status`
   - `hermes_read_memory`
   - `hermes_analyze_market`
   - `hermes_generate_report`
3. Deixar `hermes_scan_wallet`, `hermes_detect_anomaly` e `hermes_run_backtest` para depois que os engines existirem.

## Proximo passo sugerido

O proximo passo tecnico mais seguro e criar `docs/ARCHITECTURE_REVIEW.md` com uma decisao explicita:

- manter `tradingagents/` como core atual;
- criar uma camada Hermes acima dele;
- evitar monorepo grande por enquanto;
- preparar API/MCP como adaptadores, nao como centro da regra de negocio.

Depois disso, criar `docs/NEXT_IMPLEMENTATION_PLAN.md` com tarefas pequenas, criterios de conclusao e dono sugerido: Codex, Claude ou humano.
