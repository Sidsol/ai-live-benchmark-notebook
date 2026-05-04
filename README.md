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
AIMBT_PROMPT_SUITE_PATH=data/prompt_suite.json
AIMBT_ENABLE_MLFLOW=true
AIMBT_RUN_RAGAS_EVALS=false
AIMBT_RUN_DSPY_EVALS=false
```

Use provider API model IDs that your account can access. If a model returns `404`, `model_not_found`, or an unsupported-parameter error, update the corresponding `AIMBT_*_MODEL_NAME` value.

## Run

```powershell
uv run jupyter lab notebooks\live_model_benchmark.ipynb
```

Run all cells. Because this notebook is live-only, every selected prompt is sent to each configured provider and may incur provider costs. Set `AIMBT_LIVE_PROMPT_CASE_LIMIT=3` while testing, or `all` to run the full configured prompt suite.

## Prompt suite

The shared test prompt suite lives outside the notebook at `data\prompt_suite.json`. It currently includes 17 first-party cases adapted from the larger `ai-model-benchmark-tool` starter suite:

- coding prompts
- reasoning prompts
- hybrid coding/reasoning prompts
- synthetic benchmark-aligned prompts for SWE-bench Verified, SWE-bench Pro, HumanEval / LiveCodeBench, AIME / ARC-AGI-2, and GPQA Diamond

The benchmark-aligned prompts are local surrogate tasks, not official benchmark questions. To use a different prompt suite, set:

```env
AIMBT_PROMPT_SUITE_PATH=data/my_prompt_suite.json
```

Each prompt case must include `prompt_case_id`, `category`, `benchmark_refs`, `prompt`, and non-empty `expected_terms`.

## Evaluation scope

The notebook uses a lightweight text-chat evaluation flow:

- Runs each candidate model against the same shared prompt suite.
- Records latency, success/error status, response previews, raw responses, and token usage when providers return it.
- Scores output quality with a simple expected-term rubric for fast automated observations.
- Combines live prompt quality, public benchmark evidence, reliability, latency fit, and risk adjustment into a weighted decision matrix.

The quality score is not a formal correctness proof. It is a classroom-friendly automated observation that can be strengthened with task-specific graders and human review. Neuro-symbolic validation and formal verification are outside this simple notebook's scope.

The public benchmark evidence table includes editable source notes. Replace the placeholder scores and notes with the current cited sources you use for your submission.

## Framework integrations

The notebook includes lightweight support for MLflow, RAGAS, and DSPy:

| Framework | Default | What it does |
| --- | --- | --- |
| MLflow | Enabled | Logs local experiment params, metrics, decision weights, invocation summaries, quality scores, benchmark evidence, decision matrix, and risk review to `mlruns\`. |
| RAGAS | Disabled | Optionally evaluates successful model answers with RAGAS metrics using prompt/reference context. This can make extra evaluator LLM calls. |
| DSPy | Disabled | Optionally runs a DSPy prompt program against selected prompts and scores those outputs with the same expected-term rubric. This makes extra LLM calls. |

Enable optional framework calls in `.env` only when you want the extra cost:

```env
AIMBT_RUN_RAGAS_EVALS=true
AIMBT_RAGAS_EVAL_LIMIT=3
AIMBT_RUN_DSPY_EVALS=true
AIMBT_DSPY_EVAL_LIMIT=3
AIMBT_DSPY_MODEL_NAME=openai/gpt-5.5
```

MLflow is local by default and the notebook only uses experiment tracking plus plain artifact files. It does not register models or call MLflow Model Registry APIs, which avoids registry errors on local file-based tracking. To change the tracking location:

```env
AIMBT_MLFLOW_TRACKING_URI=file:///D:/Repos/ai-live-benchmark-notebook/mlruns
AIMBT_MLFLOW_EXPERIMENT_NAME=ai-live-benchmark-notebook
```

