# AI Live Benchmark Notebook

Standalone, live-only Jupyter notebook for comparing OpenAI, Anthropic, and Google models. The notebook keeps all benchmark logic in one file and displays results directly in notebook cells; it does not use mocks or write a separate report.

## Setup

```powershell
uv sync
Copy-Item .env.example .env
```

Edit `.env` and add:

| Provider | Variable |
| --- | --- |
| OpenAI | `OPENAI_API_KEY` |
| Anthropic | `ANTHROPIC_API_KEY` |
| Google | `GOOGLE_API_KEY` |

Optional model ID overrides use the same names as the previous tool:

```env
AIMBT_OPENAI_MODEL_NAME=gpt-5.5
AIMBT_ANTHROPIC_MODEL_NAME=claude-opus-4-7
AIMBT_GOOGLE_MODEL_NAME=gemini-3.1
AIMBT_LIVE_PROMPT_CASE_LIMIT=all
```

Use provider API model IDs that your account can access. If a model returns `404`, `model_not_found`, or an unsupported-parameter error, update the corresponding `AIMBT_*_MODEL_NAME` value.

## Run

```powershell
uv run jupyter lab notebooks\live_model_benchmark.ipynb
```

Run all cells. Because this notebook is live-only, every selected prompt is sent to each configured provider and may incur provider costs. Set `AIMBT_LIVE_PROMPT_CASE_LIMIT=3` while testing, or `all` to run the full built-in prompt suite.

