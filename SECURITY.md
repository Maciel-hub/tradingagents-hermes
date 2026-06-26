# SECURITY.md — Política de Segurança

Data: 2026-06-26

---

## Escopo

Este documento define os limites de segurança para desenvolvimento, colaboração
e futura operação do projeto TradingAgents / Hermes.

Aplica-se a todos os colaboradores humanos e agentes (Claude Code, Codex).

---

## Regras absolutas (nunca violar)

### Secrets e credenciais

- Nunca commitar API keys, tokens, passwords, private keys ou secrets reais
- Nunca incluir valores reais em arquivos de exemplo, docs ou comentários de código
- `.env`, `.env.*` (exceto `.env.example` e `.env.enterprise.example` com valores vazios) nunca entram no repositório
- Se um secret for acidentalmente commitado: alertar imediatamente o usuário, revogar a credencial, fazer commit de remoção

### Trade real

- Nunca executar ordens reais em exchanges sem aprovação humana explícita
- Trade real nunca automático — sempre exige confirmação manual antes de qualquer execução
- Nenhum agente (Claude, Codex, Hermes) tem autorização para iniciar trade real sozinho
- Logs obrigatórios para toda ação que interaja com exchange real

### Wallets e private keys

- Nunca manipular private keys de carteiras
- Wallet scanner deve ser somente leitura e nunca assinar transações
- Nenhuma função de "send", "transfer" ou "execute" sem aprovação e auditoria

### APIs externas

- Nenhum endpoint real de exchange integrado sem decisão arquitetural documentada
- Chaves de API externas armazenadas apenas em variáveis de ambiente, nunca em código
- Requests a APIs externas com autenticação devem logar audit trail (sem expor a key)

---

## Modos de operação

### Study (atual)

Uso: pesquisa, simulação, análise, leitura de memória, relatórios
Permissão: livre para leitura e geração de artefatos
Restrição: sem execução real de ordens

### Development

Uso: implementar funcionalidades, rodar testes, criar API local, criar MCP local
Permissão: alterações controladas em código, sem secrets reais, sem trade real
Restrição: mudanças que afetam runtime exigem aprovação

### Production (futuro)

Uso: ambiente de uso estável, monitoramento, relatórios confiáveis
Permissão: leitura e execução limitada
Restrição: mudanças exigem aprovação humana

### Real Trade (futuro, não ativo)

Uso: somente com controles formais, aprovação e auditoria
Permissão: nunca automático
Restrição: private keys nunca expostas a agentes; logs obrigatórios

---

## Fronteiras de segurança para API/MCP (quando existirem)

- API local não expõe secrets, private keys ou endpoints de trade real
- MCP não acessa secrets diretamente — usa API/Control Plane como intermediário
- Ferramentas MCP iniciais devem ser somente leitura/análise
- Ferramentas `hermes_execute_trade`, `hermes_scan_wallet` e similares
  ficam fora do escopo até controles formais existirem

---

## Gestão de dependências

- Não adicionar dependências sem justificativa documentada
- Não regenerar `uv.lock` sem plano e aprovação para mudanças relevantes
- Dependências com CVE conhecida devem ser removidas ou atualizadas imediatamente
  (histórico: `chainlit` removido por CVE — ver `CHANGELOG.md`)

---

## Relatórios e artefatos gerados

- `reports/` é ignorado pelo `.gitignore` — nunca versionar relatórios gerados em runtime
- Artefatos de análise com dados sensíveis (posições, saldos, chaves) não devem ser
  salvos em disco sem criptografia ou permissões adequadas

---

## Auditoria de execuções (futuro)

Quando Hermes Control Plane existir, toda execução sensível deve registrar:

- `request_received`
- `permission_checked`
- `engine_started` / `engine_completed`
- `blocked_sensitive_action`
- `human_approval_required`

---

## Reporte de vulnerabilidades

Se encontrar uma vulnerabilidade de segurança neste projeto:

1. Não commitar a descoberta em histórico público
2. Reportar diretamente ao responsável pelo repositório
3. Aguardar orientação antes de qualquer ação

---

## Referências

- `AGENTS.md` — regras gerais para agentes
- `CLAUDE.md` — instruções específicas para Claude Code
- `docs/ARCHITECTURE_REVIEW.md` — decisões arquiteturais de segurança
- `docs/NEXT_IMPLEMENTATION_PLAN.md` — P1.3 e checklist pré-MCP
