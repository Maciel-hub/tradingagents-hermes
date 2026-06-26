# LLM Clients - Consistency Improvements

## Pendente

### 1. `validate_model()` nunca é chamada
- `validators.py` define `validate_model(provider, model) -> bool`, mas
  nenhum arquivo a importa ou chama (confirmado em 2026-06-26).
- `create_llm_client()` em `factory.py` não valida o nome do modelo antes
  de instanciar o cliente — modelos desconhecidos chegam à SDK sem aviso.
- Ação sugerida: adicionar chamada em `create_llm_client()` com `warnings.warn`
  (não erro) para modelos não reconhecidos, mantendo retrocompatibilidade.
- Nota: URLs padrão de Qwen diferem entre `cli/utils.py`
  (`dashscope.aliyuncs.com`) e `openai_client.py` (`dashscope-intl`).
  Alinhar antes ou junto desta tarefa.

## Concluído

### ~~Inconsistent parameter handling~~ (Fixed)
- GoogleClient now accepts unified `api_key` and maps it to `google_api_key`

### ~~`base_url` accepted but ignored~~ (Fixed)
- All clients now pass `base_url` to their respective LLM constructors

### ~~Update validators.py with models from CLI~~ (Fixed)
- Synced in v0.2.2
