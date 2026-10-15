# ledgermem-eval

Open-source benchmark harness and public leaderboard for LLM memory systems.

This repo contains two complementary parts:

- **`src/`, `tests/`, `harness/`** — a Python harness that runs a set of
  reproducible benchmarks (LongMemEval, LoCoMo, RealAgentMem) against a
  pluggable set of memory systems (LedgerMem, Mem0, Zep, OpenAI Assistants,
  baseline).
- **`web/`** — a Next.js leaderboard at
  [eval.proofly.dev](https://eval.proofly.dev) that pulls published runs from
  a public R2 bucket and renders sortable tables and Pareto charts.

## Why another benchmark?

Memory benchmarks for LLM agents are still young, and most published numbers
are produced by the system's own authors with no third-party reproduction. We
care about three properties:

1. **Reproducibility.** Pin the harness, pin the model, pin the dataset.
2. **Latency honesty.** Cloud systems do not get to strip the network.
3. **Open submissions.** Any new memory system can land an adapter via PR.

## Quick reproduction

Using the harness directly:

```bash
pip install -e ".[dev]"
getmnemo-eval list-benchmarks
getmnemo-eval list-systems
getmnemo-eval run --benchmark longmemeval --mock --output ./results
```

Against real systems (`--systems` takes a comma-separated list from
`getmnemo-eval list-systems`, or `all`):

```bash
export GETMNEMO_API_KEY=...
export GETMNEMO_WORKSPACE_ID=...
export MEM0_API_KEY=...
export ZEP_API_KEY=...
export OPENAI_API_KEY=...
getmnemo-eval run --benchmark longmemeval --systems all --output ./results
```

Or build the pinned image from the bundled `Dockerfile` (its entrypoint is
`getmnemo-eval`):

```bash
docker build -t getmnemo-eval .
docker run --rm \
  -e GETMNEMO_API_KEY=... \
  -e GETMNEMO_WORKSPACE_ID=... \
  -e MEM0_API_KEY=... \
  -e ZEP_API_KEY=... \
  -e OPENAI_API_KEY=... \
  -v "$PWD/results:/work/results" \
  getmnemo-eval run --benchmark longmemeval --systems all --output /work/results
```

### Environment variables

| Variable | Used by | Notes |
| --- | --- | --- |
| `GETMNEMO_API_KEY` | `getmnemo` adapter | Required for that adapter. |
| `GETMNEMO_WORKSPACE_ID` | `getmnemo` adapter | Required for that adapter. |
| `GETMNEMO_BASE_URL` | `getmnemo` adapter | Optional API base URL override. |
| `MEM0_API_KEY` | `mem0` adapter | |
| `MEM0_USER_ID` | `mem0` adapter | Optional; defaults to `getmnemo-eval`. |
| `ZEP_API_KEY` | `zep` adapter | |
| `OPENAI_API_KEY` | `openai-assistants` adapter | |
| `GETMNEMO_EVAL_SEED` | `run` | Optional integer seed. |

## Layout

```
.
├── src/ledgermem_eval/        # Python harness
│   ├── adapters/              # one file per memory system
│   ├── benchmarks/            # one file per benchmark
│   ├── scoring.py             # recall, precision, F1, latency percentiles
│   ├── output.py              # JSON result serialization
│   └── cli.py                 # getmnemo-eval CLI (click)
├── tests/                     # pytest unit + smoke tests
├── harness/                   # docs and reproducibility artefacts
├── web/                       # Next.js leaderboard site
├── Dockerfile                 # pinned image for reproducible runs
└── .github/workflows/         # CI + scheduled bench runs
```

## Submitting a system

1. Add `src/ledgermem_eval/adapters/<name>_adapter.py` subclassing `Adapter`.
2. Register it in `src/ledgermem_eval/adapters/__init__.py`.
3. Open a PR — CI will smoke-test the adapter against the bundled fixtures.

See `harness/README.md` for the full submission guide and the rules in
`web/app/methodology/page.tsx`.

## License

Apache 2.0 for code, CC-BY 4.0 for benchmark fixtures and prose.
