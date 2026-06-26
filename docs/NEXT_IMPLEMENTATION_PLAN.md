# Plano de Implementacao Incremental

Data: 2026-06-26

Base:

- `docs/AUDIT_CURRENT_STATE.md`
- `docs/ARCHITECTURE_REVIEW.md`

Objetivo: transformar a revisao arquitetural em uma sequencia pratica de trabalho, preservando o core atual e preparando o Hermes sem criar API/MCP cedo demais.

## Decisao operacional

Fluxo de trabalho adotado:

- Claude Code: implementador principal, faz smoke test rapido e revisa seguranca da propria mudanca.
- Codex: auditor, testador, validador, documentador e coordenador tecnico.
- Usuario: aprova decisoes criticas, especialmente arquitetura, API/MCP, secrets, wallets, exchanges e trade real.

Regra principal:

- Nada de MCP, API local, wallet, blockchain engine ou trade real antes de checkpoint, higiene de ambiente, governanca e contratos Hermes.

## Definicoes de risco e complexidade

Risco:

- Baixo: documentacao, pequenos ajustes sem alterar comportamento.
- Medio: altera fluxo de instalacao, dependencias, organizacao ou comandos.
- Alto: altera runtime, grafo, agentes, memoria, API, seguranca ou integracoes externas.

Complexidade:

- Baixa: ate algumas horas, escopo claro.
- Media: exige coordenacao entre arquivos ou testes.
- Alta: exige arquitetura, migracao, contratos ou varias etapas.

## Prioridade 0 - Checkpoint e estado seguro

### P0.1 - Criar checkpoint Git

Objetivo:

- Permitir comparacao, auditoria e reversao segura antes de qualquer codigo novo.

Arquivos provaveis:

- `.git/`
- todos os arquivos atuais versionaveis.

Risco:

- Medio, porque pode incluir artefatos locais se `git add .` for feito sem revisao.

Complexidade:

- Baixa.

Criterio de conclusao:

- `git status` funciona.
- Existe commit inicial.
- `reports/`, caches e arquivos locais sensiveis nao entram por acidente.

Dono recomendado:

- Usuario ou Claude.

Papel do Codex:

- Auditar `git status` depois.

Aprovacao humana:

- Sim.

### P0.2 - Confirmar estrategia de trabalho paralelo

Objetivo:

- Evitar Codex e Claude alterando os mesmos arquivos ao mesmo tempo.

Arquivos provaveis:

- Nenhum, salvo anotacao em docs se necessario.

Risco:

- Baixo.

Complexidade:

- Baixa.

Criterio de conclusao:

- Claude sabe quais arquivos vai editar.
- Codex sabe quais arquivos vai auditar/testar.
- Mudancas grandes ficam serializadas.

Dono recomendado:

- Usuario.

Papel do Codex:

- Manter o plano e validar escopo.

Aprovacao humana:

- Sim.

## Prioridade 1 - Governanca minima

### P1.1 - Criar AGENTS.md

Objetivo:

- Definir regras gerais para agentes no projeto.

Arquivos provaveis:

- `AGENTS.md`

Conteudo esperado:

- Papel do Hermes.
- Papel do Claude Code.
- Papel do Codex.
- Regras para nao tocar em secrets.
- Regras para trade real.
- Fluxo de teste e revisao.

Risco:

- Baixo.

Complexidade:

- Baixa.

Criterio de conclusao:

- Documento existe.
- Reflete o fluxo atual: Claude implementa, Codex audita/testa.
- Nao contradiz `ARCHITECTURE_REVIEW.md`.

Dono recomendado:

- Codex.

Papel do Claude:

- Revisar se as instrucoes combinam com o fluxo de coding dele.

Aprovacao humana:

- Sim, por ser documento de governanca.

### P1.2 - Criar CLAUDE.md

Objetivo:

- Dar instrucoes especificas para o Claude implementar com seguranca.

Arquivos provaveis:

- `CLAUDE.md`

Conteudo esperado:

- Claude como implementador principal.
- Fazer mudancas pequenas.
- Fazer smoke test.
- Marcar riscos de seguranca.
- Nao mexer em secrets/private keys.
- Nao implementar API/MCP sem plano.

Risco:

- Baixo.

Complexidade:

- Baixa.

Criterio de conclusao:

- Documento existe.
- Orienta Claude de forma pratica.
- Bate com `AGENTS.md`.

Dono recomendado:

- Codex ou Claude.

Papel do Codex:

- Auditar coerencia.

Aprovacao humana:

- Sim.

### P1.3 - Criar SECURITY.md

Objetivo:

- Registrar limites de seguranca antes de API, MCP, wallet ou exchange.

Arquivos provaveis:

- `SECURITY.md`

Conteudo esperado:

- Nunca commitar API keys.
- Nunca manipular private keys.
- Trade real sempre exige aprovacao humana.
- Separacao entre study, development, production e real_trade.
- Logs de auditoria obrigatorios para acoes sensiveis.
- MCP nao acessa secrets direto.

Risco:

- Baixo.

Complexidade:

- Baixa.

Criterio de conclusao:

- Documento existe.
- Define limites para wallet, exchange, API externa, MCP e trade real.
- Claude valida como revisor principal de seguranca.

Dono recomendado:

- Claude.

Papel do Codex:

- Revisar coerencia e checklist.

Aprovacao humana:

- Sim.

## Prioridade 2 - Higiene de ambiente e projeto

### P2.1 - Alinhar `uv.lock` com `pyproject.toml`

Objetivo:

- Remover divergencia entre versao declarada e lock.

Arquivos provaveis:

- `uv.lock`
- possivelmente `pyproject.toml`, se a decisao for ajustar dependencias antes de regenerar.

Contexto:

- `pyproject.toml` declara `0.2.4`.
- `uv.lock` declara pacote local `0.2.0`.
- `uv.lock` ainda referencia `chainlit` e `rank-bm25`.

Risco:

- Medio.

Complexidade:

- Media.

Criterio de conclusao:

- Lock reflete dependencias atuais.
- `chainlit` e `rank-bm25` nao aparecem como dependencias do pacote local se realmente removidos.
- Instalacao limpa funciona.
- Testes unitarios principais passam.

Dono recomendado:

- Claude.

Papel do Codex:

- Rodar auditoria de diff.
- Rodar testes depois.

Aprovacao humana:

- Nao obrigatoria, salvo mudanca de dependencia relevante.

### P2.2 - Atualizar ou remover TODO obsoleto de LLM clients

Objetivo:

- Eliminar documentacao interna falsa ou desatualizada.

Arquivos provaveis:

- `tradingagents/llm_clients/TODO.md`

Risco:

- Baixo.

Complexidade:

- Baixa.

Criterio de conclusao:

- TODO nao afirma que `validate_model()` nunca e chamado.
- Documento vira backlog real ou e removido.

Dono recomendado:

- Claude.

Papel do Codex:

- Conferir se a afirmacao bate com o codigo.

Aprovacao humana:

- Nao.

### P2.3 - Decidir destino de `main.py` e `test.py`

Objetivo:

- Separar exemplos/laboratorio de entradas oficiais.

Arquivos provaveis:

- `main.py`
- `test.py`
- possivel `examples/`
- possivel `scripts/`
- `README.md`, se comandos mudarem.

Risco:

- Medio, porque pode afetar habitos atuais de execucao.

Complexidade:

- Baixa a media.

Criterio de conclusao:

- Entradas oficiais estao claras.
- Scripts soltos nao confundem onboarding.
- Comentario obsoleto `reflect_and_remember` removido ou atualizado.

Dono recomendado:

- Claude.

Papel do Codex:

- Auditar imports/comandos e rodar testes.

Aprovacao humana:

- Sim, se mover arquivos usados pelo usuario.

### P2.4 - Definir politica para `reports/`

Objetivo:

- Evitar versionar relatorios gerados ou artefatos locais por acidente.

Arquivos provaveis:

- `.gitignore`
- `reports/`

Risco:

- Baixo.

Complexidade:

- Baixa.

Criterio de conclusao:

- Decisao documentada: versionar exemplos fixos ou ignorar relatorios gerados.
- `.gitignore` reflete a decisao.

Dono recomendado:

- Claude.

Papel do Codex:

- Conferir `git status`.

Aprovacao humana:

- Sim.

## Prioridade 3 - Contratos Hermes antes de codigo pesado

### P3.1 - Documentar `HermesRequest`

Objetivo:

- Definir entrada padrao para qualquer motor do Hermes.

Arquivos provaveis:

- `docs/HERMES_CONTRACTS.md`

Campos sugeridos:

- `intent`
- `mode`
- `subject`
- `date_range`
- `risk_level`
- `approval_required`
- `metadata`

Risco:

- Baixo.

Complexidade:

- Baixa.

Criterio de conclusao:

- Documento descreve campos, exemplos e validacoes.
- Inclui exemplos para `analyze_market` e `read_memory`.

Dono recomendado:

- Codex.

Papel do Claude:

- Revisar se e implementavel.

Aprovacao humana:

- Sim, por virar contrato.

### P3.2 - Documentar `HermesResult`

Objetivo:

- Definir saida padrao para API/MCP/Desktop no futuro.

Arquivos provaveis:

- `docs/HERMES_CONTRACTS.md`

Campos sugeridos:

- `status`
- `summary`
- `artifacts`
- `warnings`
- `audit_id`
- `next_actions`
- `raw_ref`

Risco:

- Baixo.

Complexidade:

- Baixa.

Criterio de conclusao:

- Documento mostra saida para sucesso, erro recuperavel e bloqueio de seguranca.

Dono recomendado:

- Codex.

Papel do Claude:

- Revisar implementabilidade.

Aprovacao humana:

- Sim.

### P3.3 - Definir eventos de auditoria Hermes

Objetivo:

- Padronizar como Hermes registra execucoes.

Arquivos provaveis:

- `docs/HERMES_CONTRACTS.md`
- futuro `hermes/memory/` ou `hermes/audit/`

Eventos sugeridos:

- `request_received`
- `permission_checked`
- `engine_started`
- `engine_completed`
- `artifact_written`
- `human_approval_required`
- `blocked_sensitive_action`

Risco:

- Baixo.

Complexidade:

- Media.

Criterio de conclusao:

- Eventos documentados com campos minimos.
- Nao depende ainda de banco/API.

Dono recomendado:

- Codex.

Papel do Claude:

- Revisar viabilidade tecnica.

Aprovacao humana:

- Sim.

## Prioridade 4 - Primeira implementacao Hermes minima

### P4.1 - Criar pacote/camada Hermes minima

Objetivo:

- Introduzir `hermes/` sem tocar no core `tradingagents/`.

Arquivos provaveis:

- `hermes/__init__.py`
- `hermes/schemas.py`
- `hermes/control_plane/__init__.py`
- `hermes/engines/__init__.py`
- `tests/test_hermes_contracts.py`

Risco:

- Medio.

Complexidade:

- Media.

Criterio de conclusao:

- Pacote importa sem side effects.
- Schemas basicos existem.
- Testes unitarios passam.
- Nenhum comportamento do `tradingagents/` muda.

Dono recomendado:

- Claude.

Papel do Codex:

- Rodar testes.
- Auditar acoplamento.

Aprovacao humana:

- Sim, por criar nova estrutura.

### P4.2 - Criar Hermes Market Engine Adapter

Objetivo:

- Permitir que Hermes chame `TradingAgentsGraph` por uma fronteira controlada.

Arquivos provaveis:

- `hermes/engines/market.py`
- `hermes/schemas.py`
- `tests/test_hermes_market_adapter.py`

Risco:

- Alto, porque toca fluxo de execucao se chamar grafo real.

Complexidade:

- Media a alta.

Criterio de conclusao:

- Adapter aceita `HermesRequest`.
- Adapter monta config sem chamar CLI.
- Teste com mock de `TradingAgentsGraph` passa.
- Nao executa LLM real em teste unitario.

Dono recomendado:

- Claude.

Papel do Codex:

- Testar com mocks.
- Verificar que nao ha chamada real de LLM/dados em unit tests.

Aprovacao humana:

- Sim.

### P4.3 - Criar Hermes Control Plane minimo

Objetivo:

- Centralizar validacao de modo e roteamento para engines.

Arquivos provaveis:

- `hermes/control_plane/service.py`
- `hermes/safety.py`
- `tests/test_hermes_control_plane.py`

Risco:

- Medio.

Complexidade:

- Media.

Criterio de conclusao:

- `study` permite `analyze_market`.
- `real_trade` bloqueia qualquer intent de execucao.
- Intents desconhecidas retornam erro controlado.
- Testes cobrem permissao basica.

Dono recomendado:

- Claude.

Papel do Codex:

- Validar regras e testes.

Aprovacao humana:

- Sim.

## Prioridade 5 - Testes e validacao

### P5.1 - Rodar suite atual

Objetivo:

- Confirmar que mudancas nao quebraram TradingAgents.

Arquivos provaveis:

- Nenhum, salvo caches gerados por pytest.

Comando provavel:

```powershell
python -m pytest
```

Risco:

- Baixo.

Complexidade:

- Baixa a media, depende de ambiente.

Criterio de conclusao:

- Suite passa ou falhas sao documentadas.
- Falhas por ambiente sao separadas de falhas de codigo.

Dono recomendado:

- Codex.

Papel do Claude:

- Corrigir falhas reais.

Aprovacao humana:

- Nao.

### P5.2 - Criar checklist de validacao pre-MCP

Objetivo:

- Definir travas antes de expor ferramentas externas.

Arquivos provaveis:

- `docs/PRE_MCP_CHECKLIST.md`

Itens minimos:

- Git limpo.
- Tests passando.
- Sem secrets no repo.
- API/servico interno estavel.
- Permissoes testadas.
- Logs de auditoria definidos.
- Nenhuma ferramenta de trade real exposta.

Risco:

- Baixo.

Complexidade:

- Baixa.

Criterio de conclusao:

- Checklist existe.
- Pode ser usado por Codex antes de liberar MCP.

Dono recomendado:

- Codex.

Papel do Claude:

- Revisar itens de seguranca.

Aprovacao humana:

- Sim.

## Prioridade 6 - API local

### P6.1 - Desenhar API local

Objetivo:

- Definir endpoints antes de implementar servidor.

Arquivos provaveis:

- `docs/API_LOCAL_PLAN.md`

Endpoints iniciais sugeridos:

- `GET /status`
- `POST /analyze/market`
- `GET /memory`
- `GET /reports/{id}`

Risco:

- Medio.

Complexidade:

- Media.

Criterio de conclusao:

- Endpoints documentados.
- Payloads usam `HermesRequest` e `HermesResult`.
- Nao ha endpoint de trade real.

Dono recomendado:

- Codex.

Papel do Claude:

- Revisar implementabilidade.

Aprovacao humana:

- Sim.

### P6.2 - Implementar API local fina

Objetivo:

- Expor Hermes Control Plane sem chamar CLI.

Arquivos provaveis:

- `apps/api/`
- `pyproject.toml`
- testes de API

Risco:

- Alto.

Complexidade:

- Alta.

Criterio de conclusao:

- API sobe localmente.
- Endpoints seguros funcionam com mocks.
- Tests passam.
- Nao expoe secrets.

Dono recomendado:

- Claude.

Papel do Codex:

- Rodar testes.
- Testar endpoints.
- Auditar seguranca basica.

Aprovacao humana:

- Sim.

## Prioridade 7 - MCP

### P7.1 - Desenhar MCP Hermes

Objetivo:

- Definir ferramentas MCP depois da API/servico interno.

Arquivos provaveis:

- `docs/MCP_INTEGRATION.md`

Ferramentas iniciais:

- `hermes_get_status`
- `hermes_read_memory`
- `hermes_analyze_market`
- `hermes_generate_report`

Risco:

- Medio.

Complexidade:

- Media.

Criterio de conclusao:

- Ferramentas documentadas.
- Cada ferramenta mapeia para endpoint/servico seguro.
- Ferramentas sensiveis ficam explicitamente fora do escopo.

Dono recomendado:

- Codex.

Papel do Claude:

- Revisar viabilidade de implementacao.

Aprovacao humana:

- Sim.

### P7.2 - Implementar MCP Hermes minimo

Objetivo:

- Permitir que agentes externos chamem ferramentas Hermes seguras.

Arquivos provaveis:

- `tools/hermes-mcp-server/`
- `.mcp.json`
- testes de tool wrappers

Risco:

- Alto.

Complexidade:

- Alta.

Criterio de conclusao:

- MCP chama API/Control Plane, nao `TradingAgentsGraph` direto.
- Ferramentas iniciais funcionam.
- Logs de uso existem.
- Sem secrets.
- Sem trade real.

Dono recomendado:

- Claude.

Papel do Codex:

- Auditar wiring.
- Testar ferramentas.
- Rodar checklist pre-MCP.

Aprovacao humana:

- Sim.

## Prioridade 8 - Expansao futura cripto/blockchain

### P8.1 - Planejar blockchain engine

Objetivo:

- Definir engine on-chain separada do market data engine atual.

Arquivos provaveis:

- `docs/BLOCKCHAIN_ENGINE_PLAN.md`

Risco:

- Medio.

Complexidade:

- Alta.

Criterio de conclusao:

- Escopo definido.
- Fontes de dados definidas.
- Wallet scanner inicialmente somente leitura.
- Nada de private keys.

Dono recomendado:

- Codex para planejamento.
- Claude para implementacao futura.

Aprovacao humana:

- Sim.

### P8.2 - Planejar anomaly detector

Objetivo:

- Definir deteccao de anomalias sem confundir com decisao automatica de trade.

Arquivos provaveis:

- `docs/ANOMALY_DETECTION_PLAN.md`

Risco:

- Medio.

Complexidade:

- Alta.

Criterio de conclusao:

- Tipos de anomalia definidos.
- Dados necessarios definidos.
- Saida e alerta, nao ordem de trade.

Dono recomendado:

- Codex para planejamento.
- Claude para implementacao futura.

Aprovacao humana:

- Sim.

## Primeira leva recomendada

### Para o Claude agora

1. Fazer checkpoint Git ou confirmar com o usuario antes.
2. Criar `AGENTS.md`.
3. Criar `CLAUDE.md`.
4. Criar `SECURITY.md`.
5. Ajustar `tradingagents/llm_clients/TODO.md`.
6. Avaliar `uv.lock` e propor estrategia de regeneracao.

### Para o Codex agora

1. Auditar `git status` depois do checkpoint.
2. Revisar `AGENTS.md`, `CLAUDE.md` e `SECURITY.md`.
3. Validar se o plano continua coerente depois das mudancas do Claude.
4. Rodar testes quando o ambiente estiver pronto.
5. Atualizar este plano se alguma decisao mudar.

## O que nao fazer ainda

- Nao criar API local agora.
- Nao criar MCP agora.
- Nao reorganizar para monorepo grande agora.
- Nao mover `tradingagents/` para dentro de `hermes/`.
- Nao criar wallet scanner agora.
- Nao adicionar exchange connector agora.
- Nao implementar trade real.
- Nao misturar blockchain dentro de `tradingagents/dataflows/` sem plano.

## Criterio para passar da fase documental para codigo Hermes

Pode iniciar `hermes/` quando:

1. checkpoint Git existir;
2. docs de governanca existirem;
3. lock/dependencias estiverem compreendidos;
4. contratos `HermesRequest` e `HermesResult` estiverem documentados;
5. o usuario aprovar criar a camada Hermes minima;
6. houver plano de testes com mocks.

## Criterio para iniciar API/MCP

Pode iniciar API local quando:

1. Hermes Control Plane minimo existir;
2. Market Adapter existir;
3. testes unitarios passarem;
4. seguranca basica bloquear acoes sensiveis;
5. a API estiver documentada.

Pode iniciar MCP quando:

1. API local ou servico interno estiver estavel;
2. checklist pre-MCP estiver completo;
3. ferramentas MCP forem somente leitura/analise;
4. o usuario aprovar.

## Observacao final

Este plano favorece evolucao controlada. O projeto atual ja tem valor como motor de analise de mercado; o Hermes deve crescer ao redor dele primeiro, aprender a orquestrar com seguranca, e so depois expandir para MCP, blockchain, desktop e SaaS.
