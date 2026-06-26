# Revisao Arquitetural Hermes + TradingAgents

Data da revisao: 2026-06-26

Escopo: definir como o Hermes deve trabalhar sobre o projeto atual sem reescrever a base e sem implementar API/MCP ainda.

Documento de entrada: `docs/AUDIT_CURRENT_STATE.md`

## Decisao principal

O melhor caminho e tratar o Hermes como uma camada de controle, memoria operacional, seguranca e orquestracao acima do core atual.

O `tradingagents/` deve ser preservado como o primeiro motor de analise de mercado do Hermes, nao renomeado nem absorvido de forma apressada. A evolucao deve acontecer por adaptadores e fronteiras claras:

```text
Usuario / Codex / Claude / futuro Desktop
        |
        v
Hermes Interface Layer
        |
        v
Hermes Control Plane
        |
        +--> Hermes Memory and Audit
        +--> Hermes Safety and Permissions
        +--> Hermes Market Engine Adapter --> TradingAgentsGraph
        +--> Futuro Blockchain Engine
        +--> Futuro Risk Engine
        +--> Futuro Backtest Engine
        |
        v
API local / MCP server / Desktop / SaaS
```

Em termos simples: Hermes deve decidir "o que fazer, com quais limites e como registrar"; TradingAgents deve continuar executando "analise multiagente de mercado" ate que exista uma razao concreta para extrair ou substituir partes internas.

## O que deve ser mantido

### `tradingagents/graph`

Manter como core de execucao do fluxo multiagente atual.

Motivo:

- `TradingAgentsGraph` ja e uma fronteira utilizavel.
- O grafo ja organiza analistas, pesquisadores, trader, risco e portfolio manager.
- Checkpoint/resume ja existe.

Uso recomendado pelo Hermes:

- Chamar `TradingAgentsGraph` por uma camada de servico, nao diretamente por API/MCP.
- Transformar entrada e saida em contratos estaveis antes de expor para interfaces externas.

### `tradingagents/llm_clients`

Manter como camada oficial de provedores LLM.

Motivo:

- Ja suporta multiplos provedores.
- Evita espalhar configuracao de modelos pelos agentes.
- Ja tem normalizacao de conteudo e validacao de modelo.

Uso recomendado pelo Hermes:

- Hermes deve definir politicas de modelo e custo por modo de operacao.
- A implementacao de provedores deve continuar isolada nesta camada.

### `tradingagents/dataflows`

Manter como market data engine inicial.

Motivo:

- Ja existe roteamento de vendors.
- Ja existe suporte yfinance e Alpha Vantage.
- Ja ha preocupacao com look-ahead bias.

Uso recomendado pelo Hermes:

- Tratar como fonte de dados de mercado tradicional.
- Nao misturar blockchain/on-chain aqui sem uma decisao explicita.

### `tradingagents/agents/utils/memory.py`

Manter como memoria especifica de decisoes de trading.

Motivo:

- O log append-only atual e util.
- Ha reflexao diferida baseada em retorno realizado.

Uso recomendado pelo Hermes:

- Nao transformar esta memoria em memoria geral do Hermes.
- Criar uma memoria operacional separada para estudos, hipoteses, auditoria e decisoes de projeto.

### Testes existentes

Manter e expandir.

Motivo:

- Cobrem memoria, structured output, checkpoint, rating e validacao de modelo.
- Sao uma rede de seguranca para qualquer refatoracao futura.

## O que deve ser reorganizado

### CLI

`cli/main.py` deve continuar funcionando, mas nao deve virar dependencia da API ou do MCP.

Problema:

- Mistura UI, streaming, persistencia, status de agentes e execucao.

Direcao:

- Extrair no futuro uma camada de servico que a CLI, a API e o MCP possam usar.
- Manter a CLI como interface, nao como core.

### Scripts soltos

`main.py` e `test.py` parecem material de laboratorio.

Direcao:

- Mover para `examples/` ou `scripts/` em uma fase futura.
- Atualizar comentarios obsoletos como `reflect_and_remember`.

### Dependencias

`uv.lock` esta divergente do `pyproject.toml`.

Direcao:

- Regenerar ou alinhar o lock antes de mexer em API/MCP.
- Confirmar remocao de dependencias antigas como `chainlit` e `rank-bm25`.

### Documentacao de agentes

Faltam `AGENTS.md` e `CLAUDE.md`.

Direcao:

- Criar instrucoes claras para Codex e Claude.
- Definir que ambos sao colaboradores de desenvolvimento, nao componentes runtime do Hermes.

## Arquitetura recomendada

### Camada 1 - Core existente

Nome recomendado por enquanto:

- `tradingagents/`

Responsabilidade:

- Analise multiagente de ativos financeiros.
- Execucao do grafo.
- Dados de mercado tradicionais.
- Memoria de decisoes de trading.

Nao deve ser responsavel por:

- API publica/local.
- MCP.
- Desktop.
- SaaS.
- Wallets.
- Private keys.
- Orquestracao geral do Hermes.

### Camada 2 - Hermes Control Plane

Nome futuro possivel:

- `hermes/`
- ou `hermes_core/`

Responsabilidade:

- Receber intencoes do usuario.
- Validar permissao e modo de operacao.
- Decidir qual engine chamar.
- Registrar execucoes.
- Unificar memoria operacional.
- Normalizar respostas para interfaces.

Exemplo de contrato conceitual:

```text
HermesRequest
- intent: analyze_market | read_memory | generate_report | scan_wallet | run_backtest
- mode: study | development | production | real_trade
- subject: ticker, wallet, protocol, market, strategy
- date_range
- risk_level
- approval_required

HermesResult
- status
- summary
- artifacts
- warnings
- audit_id
- next_actions
```

### Camada 3 - Adapters de engines

Nome futuro possivel:

- `hermes/engines/market.py`
- `hermes/engines/blockchain.py`
- `hermes/engines/risk.py`
- `hermes/engines/backtest.py`

Responsabilidade:

- Adaptar o pedido Hermes para engines concretas.
- Esconder detalhes internos de cada engine.

Primeiro adapter recomendado:

- `HermesMarketEngineAdapter`

Responsabilidade:

- Receber pedido Hermes de analise de mercado.
- Construir config para `TradingAgentsGraph`.
- Chamar `propagate(ticker, trade_date)`.
- Transformar resultado em `HermesResult`.

### Camada 4 - Interfaces

Interfaces futuras:

- CLI atual.
- API local.
- MCP server.
- Desktop terminal.
- SaaS.

Regra:

- Interfaces chamam Hermes Control Plane.
- Interfaces nao chamam diretamente `TradingAgentsGraph`, dataflows, wallets ou exchanges.

## Como o Hermes deve trabalhar no projeto

### Papel do Hermes

Hermes deve ser o "operador inteligente" do projeto:

- entende o objetivo do usuario;
- escolhe o motor correto;
- aplica politicas de seguranca;
- registra o que aconteceu;
- aprende com resultados;
- gera relatorios e proximos passos;
- mantem contexto entre execucoes.

### Papel do TradingAgents

TradingAgents deve ser o motor de analise de mercado:

- recebe ticker/data/config;
- executa agentes financeiros;
- gera decisao e relatorios;
- registra memoria de trading;
- retorna sinal final.

### Papel do Claude Code

Claude Code deve ser usado como implementador principal:

- codar mudancas incrementais;
- fazer refatoracoes localizadas;
- criar ou ajustar modulos;
- fazer smoke tests rapidos da propria mudanca;
- revisar seguranca de implementacoes sensiveis;
- apontar riscos de arquitetura quando estiver codando.

Claude Code nao deve:

- implementar trade real sem aprovacao;
- manipular secrets ou private keys;
- misturar mudancas grandes de arquitetura com feature code sem plano;
- considerar smoke test proprio como validacao final.

### Papel do Codex

Codex deve ser usado como auditor, validador e coordenador tecnico:

- auditoria estrutural;
- revisao do estado atual do repositorio;
- criacao e manutencao de documentacao tecnica;
- desenho de planos incrementais;
- execucao de testes mais completos;
- verificacao de regressao;
- organizacao de criterios de conclusao;
- checagem de coerencia entre codigo, docs, configs e roadmap.

Codex tambem pode implementar quando necessario, mas neste fluxo do projeto o papel preferencial e testar, validar, documentar e orientar os proximos passos depois das mudancas do Claude.

Codex nao deve:

- decidir sozinho estrategia critica;
- aprovar trade real sem aprovacao humana;
- mexer em secrets;
- sobrescrever mudancas do Claude sem comparar contexto;
- tratar revisao documental como substituto de testes.

### Fluxo Claude + Codex

Fluxo recomendado para trabalho paralelo:

1. Claude implementa uma mudanca pequena e faz smoke test.
2. Codex audita o diff ou o estado resultante.
3. Codex roda ou propoe testes mais completos.
4. Claude corrige achados de implementacao.
5. Codex atualiza documentacao/plano quando a decisao tecnica muda.
6. Usuario aprova mudancas criticas.

Claude Code nao precisa conversar diretamente com Codex. Ambos devem trabalhar sobre os mesmos documentos, checkpoints e criterios de conclusao.

### Papel do usuario

O usuario aprova:

- mudancas de arquitetura;
- criacao de API/MCP;
- integracoes com exchanges;
- qualquer passo envolvendo wallet, private keys ou trade real;
- reorganizacoes grandes de pasta.

## Modos de operacao recomendados

### Study

Uso:

- pesquisa;
- simulacao;
- analise;
- leitura de memoria;
- relatorios.

Permissao:

- livre para leitura e geracao de artefatos.
- sem execucao real.

### Development

Uso:

- implementar funcionalidades;
- rodar testes;
- criar API local;
- criar MCP local.

Permissao:

- alteracoes controladas em codigo.
- sem secrets reais.
- sem trade real.

### Production

Uso futuro:

- ambiente de uso estavel;
- monitoramento;
- relatorios confiaveis.

Permissao:

- leitura e execucao limitada.
- mudancas exigem aprovacao.

### Real Trade

Uso futuro, nao recomendado agora.

Permissao:

- nunca automatico no inicio.
- sempre com aprovacao humana explicita.
- sem private keys expostas a agentes.
- logs obrigatorios.

## Preparacao para API local

Nao criar API ainda sem concluir:

1. checkpoint Git;
2. alinhamento de dependencias;
3. contrato `HermesRequest` e `HermesResult`;
4. politica de seguranca;
5. servico interno que chama `TradingAgentsGraph`.

Quando chegar a hora, a API local deve ser fina:

```text
POST /analyze/market
GET  /status
GET  /memory
GET  /reports/{id}
```

Ela nao deve:

- chamar a CLI interativa;
- expor private keys;
- executar trades;
- permitir alteracao de estrategia sem autorizacao.

## Preparacao para MCP

MCP deve vir depois da API local ou depois de um servico interno estavel.

Ferramentas iniciais seguras:

- `hermes_get_status`
- `hermes_read_memory`
- `hermes_analyze_market`
- `hermes_generate_report`

Ferramentas que devem esperar:

- `hermes_scan_wallet`
- `hermes_detect_anomaly`
- `hermes_run_backtest`
- `hermes_execute_trade`

Motivo:

- wallet, anomalia e backtest precisam de engines proprias;
- trade real precisa de politicas e aprovacoes formais.

## Estrutura futura recomendada

Nao aplicar agora automaticamente. Esta e a direcao gradual:

```text
D:\TradingAgents-main
|-- tradingagents/              # core atual preservado
|-- cli/                        # interface CLI atual
|-- hermes/                     # futura camada de controle
|   |-- control_plane/
|   |-- engines/
|   |-- memory/
|   |-- safety/
|   `-- schemas/
|-- apps/
|   `-- api/                    # futura API local
|-- tools/
|   `-- hermes-mcp-server/      # futuro MCP
|-- docs/
|-- tests/
`-- scripts/
```

Ordem correta:

1. documentar;
2. estabilizar ambiente;
3. criar contratos;
4. criar camada Hermes minima;
5. adaptar TradingAgents como market engine;
6. criar API local;
7. criar MCP;
8. expandir para blockchain, desktop e SaaS.

## Ordem de implementacao recomendada

### Fase 0 - Checkpoint e higiene

Objetivo:

- garantir que o trabalho paralelo possa ser auditado.

Itens:

- iniciar Git ou confirmar repositorio correto;
- revisar `uv.lock`;
- remover/atualizar docs obsoletas;
- decidir destino de scripts soltos.

### Fase 1 - Governanca

Objetivo:

- alinhar Codex, Claude e humano.

Itens:

- criar `AGENTS.md`;
- criar `CLAUDE.md`;
- criar `SECURITY.md`;
- criar `docs/NEXT_IMPLEMENTATION_PLAN.md`.

### Fase 2 - Contratos Hermes

Objetivo:

- definir como Hermes fala com engines.

Itens:

- documentar `HermesRequest`;
- documentar `HermesResult`;
- definir modos de operacao;
- definir eventos de auditoria.

### Fase 3 - Hermes Market Adapter

Objetivo:

- permitir que Hermes use TradingAgents sem acoplar interface.

Itens:

- criar adapter para `TradingAgentsGraph`;
- transformar resultado do grafo em resultado Hermes;
- testar com mocks.

### Fase 4 - API local

Objetivo:

- expor funcionalidades seguras para interfaces.

Itens:

- status;
- analise de mercado;
- leitura de memoria;
- relatorios.

### Fase 5 - MCP

Objetivo:

- permitir que Codex/Claude chamem ferramentas Hermes com seguranca.

Itens:

- servidor MCP fino;
- tools somente leitura/analise;
- logs de uso;
- sem secrets.

### Fase 6 - Expansao cripto/blockchain

Objetivo:

- adicionar capacidades novas sem contaminar o market engine atual.

Itens:

- blockchain engine;
- wallet scanner somente leitura;
- anomaly detector;
- risk engine para cripto;
- backtest engine formal.

## Riscos arquiteturais

1. Renomear/reorganizar tudo cedo demais.

Impacto:

- quebra imports;
- dificulta trabalho paralelo;
- aumenta retrabalho.

Mitigacao:

- manter `tradingagents/` intacto ate existirem contratos Hermes.

2. Fazer MCP antes da camada Hermes.

Impacto:

- MCP vira dono da regra de negocio;
- ferramentas ficam acopladas a arquivos internos;
- seguranca fica reativa.

Mitigacao:

- MCP deve chamar Hermes Control Plane ou API local, nunca core direto.

3. Misturar blockchain dentro de dataflows atuais.

Impacto:

- confunde dados de mercado tradicional com dados on-chain;
- aumenta acoplamento.

Mitigacao:

- criar blockchain engine separada quando chegar a fase certa.

4. Usar Codex/Claude como runtime do produto.

Impacto:

- dependencia operacional errada;
- perda de controle sobre decisoes criticas.

Mitigacao:

- Codex e Claude sao ferramentas de desenvolvimento/revisao.
- Hermes runtime deve ser proprio.

## Conclusao

O Hermes deve trabalhar neste projeto como uma camada superior de orquestracao segura, nao como uma reescrita do TradingAgents.

Decisao recomendada:

- preservar `tradingagents/` como core de analise de mercado;
- criar uma camada Hermes minima antes de API/MCP;
- separar memoria operacional do Hermes da memoria de decisoes de trading;
- tratar API e MCP como adaptadores;
- manter Claude e Codex como colaboradores de desenvolvimento, nao como agentes runtime do produto;
- adiar blockchain/wallet/trade real ate existirem contratos, seguranca e auditoria.

Proximo documento recomendado:

- `docs/NEXT_IMPLEMENTATION_PLAN.md`

Esse plano deve transformar esta revisao em tarefas pequenas, com risco, complexidade, criterio de conclusao e dono recomendado.
