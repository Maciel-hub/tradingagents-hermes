# CLAUDE.md — Instruções para Claude Code neste repositório

Data: 2026-06-26

---

## Contexto do projeto

Este repositório contém o core **TradingAgents** — um framework multiagente Python
para análise de ativos financeiros usando LangGraph, LangChain e múltiplos provedores LLM.

Ele é a base do futuro **Hermes**, um control plane de análise e orquestração.
Hermes ainda não existe como código neste repositório.

Leia antes de qualquer sessão:
- `docs/AUDIT_CURRENT_STATE.md`
- `docs/ARCHITECTURE_REVIEW.md`
- `docs/NEXT_IMPLEMENTATION_PLAN.md`

---

## Regra obrigatória antes de qualquer alteração de arquivo

```
1. git rev-parse --is-inside-work-tree
2. git status --short
3. Se houver mudanças não commitadas: PARAR e pedir confirmação
4. Confirmar checkpoint Git antes de editar qualquer arquivo
```

Nunca pular esta sequência, mesmo para mudanças "pequenas".

---

## Princípios de implementação

- Mudanças pequenas e incrementais — uma coisa de cada vez
- Após cada mudança: smoke test rápido da própria alteração
- Mostrar diff/resumo antes de continuar para o próximo item
- Revisar segurança de mudanças sensíveis antes de commitar
- Nunca criar arquivo de plano/análise intermediário — trabalhar pelo contexto da conversa

---

## O que implementar agora (primeira leva)

Seguir a seção "Primeira leva recomendada" de `docs/NEXT_IMPLEMENTATION_PLAN.md`:

1. Checkpoint Git (feito em 2026-06-26, commit `c38a45b`)
2. Criar `AGENTS.md`
3. Criar `CLAUDE.md` (este arquivo)
4. Criar `SECURITY.md`
5. Ajustar `tradingagents/llm_clients/TODO.md`
6. Avaliar `uv.lock` e propor estratégia de regeneração

---

## O que NÃO implementar sem aprovação

- API local ou MCP
- Wallet scanner, blockchain engine, exchange connector
- Trade real de qualquer forma
- Reorganização ampla de pastas
- Qualquer mudança que afete `TradingAgentsGraph` sem plano e testes

---

## Segurança — regras absolutas

- Nunca commitar `.env`, `.env.local`, `.env.*` com valores reais
- Nunca incluir API keys, tokens, private keys ou secrets em código ou docs
- Nunca habilitar execução de trade real — mesmo em modo "dry run" sem aprovação
- Nunca criar endpoint ou função que acesse wallet ou exchange sem aprovação
- Se encontrar um secret em qualquer arquivo: parar, alertar o usuário, não commitar

---

## Estrutura do repositório — o que preservar intacto

```
tradingagents/       # core de análise — não reorganizar sem plano
cli/                 # interface CLI atual — não transformar em API
tests/               # não remover testes existentes
docs/                # docs de arquitetura — só atualizar, não deletar
```

---

## Formato de commit

```
<tipo>(<escopo>): <descrição curta em português>

<corpo opcional explicando o porquê, não o quê>
```

Tipos: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`

Nunca usar `--no-verify`. Nunca forçar push em `master`/`main`.

---

## Após cada etapa, reportar

- Resumo do que foi feito
- Arquivos alterados (lista)
- Riscos de segurança revisados
- Smoke test executado
- Próximo passo sugerido
