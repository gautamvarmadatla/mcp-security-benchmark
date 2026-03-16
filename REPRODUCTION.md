# Reproduction Instructions

## Requirements

- Python 3.11+
- `uv` package manager

```bash
pip install uv
```

## Setup

```bash
uv venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
uv pip install -r requirements.txt
cp .env.example .env
```

Or install from `pyproject.toml`:

```bash
uv pip install -e .
uv pip install pytest pytest-asyncio
```

## Run unit tests (no servers needed)

```bash
pytest tests/test_benchmark.py -v -k "not integration"
```

Expected: **39 passed**

## Run full benchmark

Start three servers in separate terminals:

```bash
# Terminal 1 — fetch server, tracing enabled (scenarios S1, S2)
EMIT_DENIAL_TRACE=1 TRACE_DIR=evidence/server_traces python servers/fetch_http_server/server.py

# Terminal 2 — auth server (scenarios S5, S6, S10, S12)
python servers/auth_http_server/server.py

# Terminal 3 — fetch server, tracing disabled (scenario S8 only)
EMIT_DENIAL_TRACE=0 TRACE_DIR=evidence/server_traces_s8 FETCH_SERVER_PORT=8003 python servers/fetch_http_server/server.py
```

Then run:

```bash
# Terminal 4
pytest tests/test_benchmark.py -v

# Or use the entry point:
python -m client
```

Or use the one-command script (starts all servers automatically):

```bash
bash scripts/run_benchmark.sh
```

## Expected results

| Metric | dual_axis | lifecycle_only | component_only |
|--------|:---------:|:--------------:|:--------------:|
| Violation Detection Rate | 100% | 100% | 100% |
| False Positive Rate | 0% | 0% | 0% |
| Localization Accuracy | **100%** | 80% | 80% |
| Evidence Completeness | 100% | 100% | 100% |

The localization gap is driven by:
- **S8**: real egress denial; naive inference maps to `scope/runtime`; GT is `observability/observability`
- **S11**: naive phase maps `HASH_MISMATCH → admission`; GT phase is `runtime`
- **S12**: naive component maps `AUTHZ_DENIED → authz`; GT component is `scope`

## Output files

After a full run, results are written to `evidence/results/`:

```
metrics_dual_axis_*.json         # per-mode scores
metrics_lifecycle_only_*.json
metrics_component_only_*.json
baseline_comparison_*.json       # all three modes side by side
scenario_outcomes_*.csv          # per-scenario predictions vs. ground truth
results_*.csv                    # full detail rows
summary_table_*.md               # paper-ready markdown table
summary_table_*.tex              # paper-ready LaTeX table
```

Sample outputs from the reference run are included in `evidence/results/`.
