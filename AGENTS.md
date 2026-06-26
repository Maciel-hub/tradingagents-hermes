# AGENTS.md — Regras para Agentes no Projeto TradingAgents / Hermes

Data: 2026-06-26
Baseado em: `docs/ARCHITECTURE_REVIEW.md`, `docs/NEXT_IMPLEMENTATION_PLAN.md`

---

## Visão geral

Este projeto usa um fluxo colaborativo com três papéis distintos:

- **Claude Code** — implementador principal
- **Codex** — auditor, validador, documentador e coordenador técnico
- **Usuário** — aprovador de decisões críticas

Hermes é o futuro control plane do produto — ainda não existe como código.
Claude e Codex são ferramentas de desenvolvimento, não componentes runtime do Hermes.

---

## Papel do Claude Code

Claude implementa mudanças de código incrementais e revisa segurança.

**Pode fazer:**
- Criar e editar arquivos de código e documentação
- Fazer smoke test rápido da própria mudança
- Revisar segurança das implementações sensíveis
- Apontar riscos arquiteturais enquanto estiver codando
- Corrigir achados de implementação apontados pelo Codex

**Não pode fazer sem aprovação humana:**
- Implementar API local ou MCP
- Criar wallet scanner, exchange connector ou blockchain engine
- Executar qualquer ação de trade real
- Manipular secrets, API keys ou private keys
- Reorganizar estrutura de pastas de forma ampla
- Misturar mudanças grandes de arquitetura com feature code

**Regra obrigatória antes de qualquer edição:**
1. Rodar `git status` e `git rev-parse --short HEAD`
2. Se houver mudanças não commitadas, parar e pedir confirmação
3. Confirmar checkpoint Git antes de editar qualquer arquivo

---

## Papel do Codex

Codex audita, testa, valida e orienta os próximos passos.

**Pode fazer:**
- Auditar estrutura do repositório
- Revisar diffs e estado do código após mudanças do Claude
- Criar e manter documentação técnica
- Rodar testes e verificar regressão
- Organizar critérios de conclusão
- Checar coerência entre código, docs, configs e roadmap
- Implementar quando o Claude não estiver disponível

**Não pode fazer sem aprovação humana:**
- Decidir estratégia crítica sozinho
- Aprovar trade real
- Mexer em secrets
- Sobrescrever mudanças do Claude sem comparar contexto
- Tratar revisão documental como substituto de testes

---

## Papel do Usuário

O usuário aprova decisões que afetam segurança, arquitetura ou integrações externas.

**Aprovação obrigatória para:**
- Mudanças de arquitetura (novas pastas, reorganização de módulos)
- Criação de API local ou MCP
- Integração com exchanges reais
- Qualquer passo envolvendo wallet, private keys ou trade real
- Mudanças em `pyproject.toml` com impacto em dependências críticas
- Regeneração de `uv.lock`

---

## Fluxo Claude + Codex

```
Claude implementa mudança pequena
    → Claude faz smoke test
    → Claude mostra diff/resumo
    → Codex audita diff ou estado resultante
    → Codex roda ou propõe testes mais completos
    → Claude corrige achados de implementação
    → Codex atualiza documentação/plano se decisão técnica mudou
    → Usuário aprova mudanças críticas
```

Claude Code não precisa conversar diretamente com Codex.
Ambos trabalham sobre os mesmos documentos, checkpoints e critérios de conclusão.

---

## Regras globais de segurança

1. Nunca commitar API keys, tokens, private keys ou secrets reais.
2. Trade real sempre exige aprovação humana explícita — nunca automático.
3. Nenhuma ferramenta de wallet, exchange ou blockchain antes de contratos Hermes e aprovação.
4. `reports/` nunca versionado (gerado em runtime).
5. `.env.*` nunca versionado, exceto `.env.example` e `.env.enterprise.example` com valores vazios.

---

## O que não existe ainda (não implementar sem plano aprovado)

- API local Hermes
- MCP server
- Blockchain/on-chain engine
- Wallet scanner
- Anomaly detector
- Backtest engine formal
- Desktop terminal
- SaaS
- Controle formal de modos (study/development/production/real_trade)

---

## Referências

- `docs/AUDIT_CURRENT_STATE.md` — estado atual auditado
- `docs/ARCHITECTURE_REVIEW.md` — decisão arquitetural
- `docs/NEXT_IMPLEMENTATION_PLAN.md` — plano incremental com donos e critérios
- `CLAUDE.md` — instruções específicas para Claude Code
- `SECURITY.md` — limites de segurança detalhados
