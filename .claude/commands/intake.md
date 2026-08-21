# Quickstart Intake — Repo to Quickstart

Take an existing GitHub repo and produce a complete, factory-standard Intel AI Quickstart. Business solution first, tech second. Output is a ready-to-push quickstart in a separate directory — the source repo is never modified.

## Usage

`/intake <github-repo-url-or-owner/name>`

## Instructions

You are the quickstart intake agent. You take a working repo and produce a complete quickstart that meets the Intel AI Quickstart Factory standards — the same structure produced by `factory/scripts/scaffold.sh`, but fully filled in rather than templated with blanks.

The gatekeeper criterion is: **does this solve a real business problem?** Purely technical showcases (benchmarks, inference configs) are weaker candidates than quickstarts that solve an industry problem (fraud detection, healthcare agent, compliance monitoring). If there's no business story, flag it before doing anything else.

### Input

The user provides a GitHub repo URL (or owner/name) as `$ARGUMENTS`. Parse it and clone the repo.

If no argument is provided, ask the user for the repo URL.

---

### Phase 1: Clone and Understand

1. Clone the source repo to `/tmp/quickstart-intake-<repo-name>-source` (remove any prior clone first). This is READ-ONLY — never modify it.
2. Read the README, any CLAUDE.md, and ALL source files to deeply understand what the repo does, how it works, what technologies it uses, what claims it makes, and what business problem it addresses.
3. Identify the repo type: container-based, script-based, Helm-based, notebook-based.
4. Create the output directory at `/tmp/quickstart-intake-<repo-name>` — this is where the complete quickstart will be built.
5. Copy all files from the source repo into the output directory as the starting point.

### Phase 2: Business Solution Assessment (GATE CHECK)

This is the first and most important check. Answer these questions:

1. **What business problem does this solve?** Not "what technology does it use" — what outcome does a user get? (e.g., "detect financial fraud", "serve AI models without cloud API costs", "classify support tickets automatically")
2. **Who is the target user?** What role, what industry?
3. **What industry does this map to?** Pick from the approved list: Automotive, Banking and securities, Broadcasting and cable, Education, Government, Health insurance payer, Healthcare provider, Insurance, Life sciences, Manufacturing, Media and IT services, Retail, Telecommunications, Transportation, Utilities, Wholesale trade
4. **Is the business value clear in the README's first 3 sentences?** A reader should understand the business problem before seeing any technical details.
5. **Can this teach someone HOW to build it?** A good quickstart isn't just "run this script" — it shows the user the underlying technology so they understand it contextually. Could someone follow this and learn to build something similar? (This is critical for showroom lab conversion later.)

Rate the business story:
- **STRONG**: Clear industry problem, obvious user persona, business value in the first paragraph
- **MODERATE**: Technical tool with implied business value that needs to be surfaced
- **WEAK**: Pure tech showcase — no business problem articulated
- **NONE**: Utility/tooling with no end-user scenario

If WEAK or NONE, flag this prominently but still proceed — reframe the business story yourself in the output README.

### Phase 3: Build the Complete Quickstart

Work in the output directory (`/tmp/quickstart-intake-<repo-name>`). For every component, either keep what already exists (if it meets the standard) or create it fully filled in. No blanks, no TODOs, no empty templates.

#### Structure

Create any missing directories and files to match:

```
├── .github/workflows/ci.yaml
├── chart/ (if applicable)
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/test-model-access.yaml
├── contracts/openapi/ (if applicable)
├── docs/images/
├── src/ (or existing source layout)
│   └── ui.py (Gradio frontend — generated in Phase 3b)
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── contracts/__init__.py
│   ├── contracts/test_openapi_compliance.py (if applicable)
│   ├── unit/__init__.py
│   ├── integration/__init__.py
│   ├── benchmarks/__init__.py
│   ├── publication/__init__.py
│   ├── publication/test_readme.py
│   ├── validation_matrix.yaml
│   ├── claim_registry.yaml
│   └── benchmark_rubric.yaml
├── .env.example (if env vars are used)
├── .gitignore
├── Makefile
├── LICENSE
└── README.md
```

Mark components N/A and skip them if they don't apply (script-only repos don't need Helm charts, OpenAPI contracts, or docker-compose).

#### Makefile

Use the factory template structure with all targets (test-contracts, test-infra, test-unit, test-integration, test-benchmarks, test-publication, test-all, audit-claims, status, build, compose-up, compose-down, lint). Adapt paths to match the repo's actual layout. The Makefile must work — paths must resolve to real files.

#### tests/validation_matrix.yaml

Use the factory template with all 6 stages. **Fill in stage_2_unit criteria** based on what you learned from reading the source code. For example, if the repo serves an LLM, the unit criteria should test that the model loads, the health endpoint responds, inference returns valid output, etc. Write real criteria — not placeholders.

#### tests/claim_registry.yaml

Scan the README and ALL source files for performance claims — any number with a unit: "Nx faster", "X ms", "X tok/s", "X%", "X GB", "X cores", latency numbers, throughput numbers, cost savings, memory requirements. Create a complete claim entry for each:

```yaml
- id: <kebab-case-id>
  value: "<exact claim text>"
  file: <file where it appears>
  line: <line number>
  source: "<cited source if any, otherwise 'Uncited'>"
  source_url: "<URL if cited>"
  verified: false
  verified_date: null
  notes: "Needs measurement on target hardware"
```

#### tests/benchmark_rubric.yaml

Use the factory template. **Fill in the benchmarks section** with concrete thresholds extracted from the claims. For example, if the README claims "~9 tok/s at 8 cores", create a benchmark entry with `max_ms` or `min_throughput` thresholds. Every claim that can be measured should have a corresponding benchmark.

#### tests/publication/test_readme.py

Use the factory template. Adapt the `REQUIRED_SECTIONS` list if the repo uses different section names that serve the same purpose (e.g., "What you need" instead of "Requirements"). The test must pass against the README you produce.

#### tests/conftest.py

Create with shared fixtures relevant to the repo. If it's a Python project, include fixtures for the main application objects. If it's script-based, include fixtures for script paths and expected outputs.

#### .github/workflows/ci.yaml

Create a CI workflow that runs `make test-contracts` and `make test-publication` on push and PR. Keep it lightweight — no cluster, no container builds, just the static checks.

#### .env.example

If the repo uses environment variables (HF_TOKEN, MODEL, API keys, ports, etc.), create an example file with placeholder values and descriptive comments. Never include real credentials.

#### docs/images/

If no architecture diagram exists, create a Mermaid diagram in the README based on your understanding of the architecture. Add a note that `architecture.png` should be generated from the Mermaid source.

#### LICENSE

If missing, create Apache 2.0.

#### .gitignore

If missing or incomplete, create with: `__pycache__/`, `*.pyc`, `.pytest_cache/`, `*.egg-info/`, `dist/`, `build/`, `.env`, `*.egg`, `venv/`, `.venv/`

#### __init__.py files

Create empty `__init__.py` in tests/, tests/unit/, tests/integration/, tests/benchmarks/, tests/contracts/, tests/publication/ for pytest discovery.

### Phase 3b: Gradio Frontend (`src/ui.py`)

Generate a Gradio UI that makes the quickstart interactive in a browser. Every quickstart gets a `src/ui.py` — this is the demo surface that showroom labs, screenshots, and presentations are built from.

**If the source repo already has a working Gradio UI**, keep it — just verify it follows the conventions below and fix any gaps.

**If it has no UI** (or uses Streamlit/Flask/bare HTML), generate `src/ui.py` from scratch based on what you learned about the repo's API surface in Phase 1.

#### Conventions (match the existing fleet)

All 35 quickstarts in the factory follow these patterns exactly:

1. **Imports**: `gradio as gr`, `httpx` for backend calls, `os` for env vars, `json` if needed
2. **Backend URL from env var**: `BACKEND_URL = os.environ.get("<APP>_URL", "http://localhost:<port>")` — name the var after the app (e.g., `SCORER_URL`, `ROUTER_URL`, `ORCHESTRATOR_URL`)
3. **Helper functions**: `_post(path, payload)` and `_get(path)` that call the backend and return JSON. Use `httpx` (not `requests`). Include `timeout=` on every call (30s for queries, 60s for heavy ops, 10s for stats/health)
4. **`gr.Blocks` layout** (never `gr.Interface`): use `gr.Blocks(title="<Quickstart Title>")` with `gr.Tab()` sections
5. **Tab structure** — build tabs based on what the app does:

   | App type | Tab 1 (primary interaction) | Tab 2+ (secondary) |
   |---|---|---|
   | Scoring/classification | Form with inputs → result display | Batch mode, Statistics |
   | RAG/Q&A | Question textbox → answer + sources | Upload docs, Document library, Pipeline stats |
   | Agent/workflow | Scenario textbox → step-by-step output | Agent registry, System stats |
   | Inference server | Prompt textbox → model response | Model info, Performance metrics |
   | Monitoring/observability | Dashboard/metrics view | Alert config, Historical data |

6. **Input widgets**: match the API's input schema — `gr.Textbox` for free text, `gr.Number` for numeric, `gr.Dropdown` for enums/categories, `gr.Slider` for ranges, `gr.Checkbox` for toggles. Pre-fill with realistic example values that trigger an interesting response (not empty defaults)
7. **Output formatting**: use `gr.HTML` for rich results (tables, colored badges), `gr.Markdown` for structured text, `gr.Textbox` for plain logs. Build result display inline in the handler function using f-strings
8. **Risk/status badges**: color-coded HTML spans for categorical results (green/yellow/orange/red for risk levels, pass/fail, etc.)
9. **AI disclaimer**: every quickstart includes a disclaimer — vary the wording per domain but always present. Place it after the primary tab content or as a footer
10. **Launch config**: always `demo.launch(server_name="0.0.0.0", server_port=7860)`
11. **Docstring**: module-level docstring describing the tabs

#### How to determine the API surface

To build the UI, you need to know what endpoints the backend exposes. Check in this order:

1. **OpenAPI spec** in `contracts/openapi/` — this is the source of truth if it exists
2. **FastAPI route definitions** — grep for `@app.get`, `@app.post`, `@router.get`, etc. in the source
3. **Flask/Express routes** — grep for `@app.route`, `router.get`, etc.
4. **README examples** — curl commands showing API usage
5. **docker-compose.yml** — port mappings and service names reveal the architecture

For each endpoint found, extract: HTTP method, path, request body schema (field names + types), response body schema. Build one UI handler function per endpoint, and group related endpoints into tabs.

If the app has no API (script-based, notebook-based), build a UI that:
- For scripts: wraps the main script execution with `subprocess.run`, showing stdout/stderr in a `gr.Textbox`
- For notebooks: extracts the key computation into a function and wraps it with Gradio inputs/outputs

#### Example skeleton

```python
"""Gradio UI for <quickstart-title>.

Provides <N> tabs:
  1. <Primary action> -- <what it does>
  2. <Secondary> -- <what it does>
  3. Statistics -- cumulative metrics
"""

import json
import os

import gradio as gr
import httpx

BACKEND_URL = os.environ.get("<APP>_URL", "http://localhost:8000")


def _post(path: str, payload: dict) -> dict:
    resp = httpx.post(f"{BACKEND_URL}{path}", json=payload, timeout=30.0)
    resp.raise_for_status()
    return resp.json()


def _get(path: str) -> dict:
    resp = httpx.get(f"{BACKEND_URL}{path}", timeout=10.0)
    resp.raise_for_status()
    return resp.json()


def primary_action(input1: str, input2: float) -> str:
    try:
        result = _post("/api/v1/<endpoint>", {"field1": input1, "field2": input2})
    except Exception as e:
        return f"<p style='color:red;'>Error: {e}</p>"
    # Format result as HTML/Markdown
    return f"..."


def get_stats() -> str:
    try:
        stats = _get("/api/v1/stats")
    except Exception as e:
        return f"Error: {e}"
    return f"**Total:** {stats['total']}\n\n**Avg Latency:** {stats['avg_ms']:.1f} ms"


with gr.Blocks(title="<Quickstart Title>") as demo:
    gr.Markdown("# <Quickstart Title>")
    gr.Markdown("<One-line description of what the demo shows.>")

    with gr.Tab("<Primary Action>"):
        with gr.Row():
            with gr.Column():
                # Input widgets with pre-filled examples
                input1 = gr.Textbox(label="...", value="<realistic example>")
                input2 = gr.Number(label="...", value=42)
                run_btn = gr.Button("<Action Verb>", variant="primary")
            with gr.Column():
                output = gr.HTML(label="Result")
        gr.Markdown("*<AI disclaimer tailored to this domain.>*")
        run_btn.click(fn=primary_action, inputs=[input1, input2], outputs=output)

    with gr.Tab("Statistics"):
        stats_btn = gr.Button("Refresh", variant="secondary")
        stats_output = gr.Markdown(label="Stats")
        stats_btn.click(fn=get_stats, outputs=stats_output)


if __name__ == "__main__":
    demo.launch(server_name="0.0.0.0", server_port=7860)
```

#### Validation

After generating `src/ui.py`, verify:
- `python -c "import ast; ast.parse(open('src/ui.py').read())"` — syntax is valid
- All endpoint paths in the UI match real endpoints in the backend source
- Input widget types match the API's expected field types
- The env var name for the backend URL is consistent with what `docker-compose.yml` or the Helm chart sets
- `gradio` and `httpx` are in `requirements.txt` or `pyproject.toml`

If `requirements.txt` exists, add `gradio>=4.0.0` and `httpx>=0.24.0` if not already present. If `pyproject.toml` exists, add them to the dependencies list.

### Phase 4: README

This is where the business story lives. The README in the output quickstart must:

- **Lead with the business problem** — the first paragraph explains what problem is being solved and for whom, before any technology is mentioned
- H1 title: <= 64 chars, starts with action verb (Deploy, Build, Accelerate, Route, Detect, Serve, Run, Optimize, Create, Scale, Orchestrate, Automate, Monitor, Secure, Analyze, Stream, Transform, Classify, Boost, Encrypt, Govern)
- Short description: <= 160 chars, immediately after H1
- Include all required sections: Table of Contents, Overview, Detailed description (with Architecture diagrams), Requirements (Minimum hardware, Minimum software, Required user permissions), Deploy (Prerequisites, Installation, Validating the deployment, Delete), Repository structure, References, Tags
- Tags section with all required keys: Title, Description, Industry, Product, Use case, Partner, Contributor org
- Industry tag from the approved list
- Tags in `- **Key:** value` format
- Architecture diagram images with descriptive alt text
- Red Hat trademark first-mention-only rule: use "Red Hat" correctly, not "RedHat" or "Redhat"
- WCAG 2.2 AA: heading levels nest correctly (no H2→H4 skips), all images have alt text, link text is descriptive (no "click here")

If the existing README is good, keep its content and add the missing sections. If the business story needs reframing, rewrite the opening paragraphs to lead with the business problem while preserving the technical content. Adapt the section structure to fit (script-only quickstarts can have a simpler Deploy section).

### Phase 5: Intel Story & Branding

- Identify the Intel value proposition from the source code and README
- Rate: None / Weak / Moderate / Strong
- If Weak or None, add appropriate Intel context to the README based on what tech the repo actually uses (CPU inference → Intel Xeon / AMX, optimization → AVX-512 / OpenVINO, etc.)
- Branding rules: "Powered by Intel" is acceptable. "Intel Xeon" / "Intel Xeon 6" are acceptable. "Gaudi" must NOT appear in titles or user-facing branding — use "Intel accelerator" instead. Technical model names like `gpt-oss-120b-gaudi` are OK in config, not in branding.

### Phase 6: Overlap Scan

Scan the `rh-ai-quickstart` GitHub org (and optionally local quickstarts) to detect overlap with existing quickstarts. This prevents publishing a quickstart that duplicates something already in the catalog.

**Step 1: Fetch existing quickstarts from the org.**

```bash
gh repo list rh-ai-quickstart --limit 200 --json name,description
```

**Step 2: Extract features from the intake quickstart.** From the README and source, identify:
- Industry (from Tags section)
- Techniques used (RAG, tool-calling, classification, scoring, agent, governance, drift-detection, multi-model, MCP, confidential-compute, speculative-decoding, etc.)
- Key technologies (vLLM, OpenVINO, Ollama, LangChain, etc.)
- Business problem keywords (fraud, observability, compliance, routing, etc.)

**Step 3: Compare against every repo in the org.** For each org repo, check:
- **Direct name match**: same repo name = already published (skip or update)
- **Keyword overlap**: compare industry, technique, and business keywords against the org repo's name and description. Score by number of matching keywords.
- **Technique pattern match**: repos solving the same problem with the same technique (e.g., two RAG quickstarts, two inference servers, two agent governance tools)

**Step 4: Score and report.**

| Overlap level | Criteria | Action |
|---|---|---|
| **DUPLICATE** | Direct name match in org | Already published — update instead of creating new |
| **HIGH** (>=3 keyword matches) | Same technique + same business problem | Flag — explain differentiation or merge |
| **MEDIUM** (2 keyword matches) | Same technique, different angle | Note — ensure README clearly differentiates |
| **LOW** (1 keyword match) | Tangential similarity | OK to proceed |
| **NONE** | No matches | Clear to publish |

Report format:

```
OVERLAP SCAN:
  Org repos scanned: N
  Direct matches: X (already published)
  High overlap: Y
  Medium overlap: Z

  [DUPLICATE] hybrid-fraud-detection — already in org
  [HIGH] mcp-federated-tools ↔ secure-tool-planner (both: MCP + agent + tools)
  [MEDIUM] edge-ai-cpu-inference ↔ llm-cpu-serving (both: CPU inference)
```

If HIGH overlaps are found, note what makes this quickstart different (Intel hardware, different technique, different industry) so the author can sharpen the differentiation in the README.

**Step 5: Local overlap** (optional). If `/Users/jkershaw/Documents/intel-quickstarts-triforce/quickstarts/` exists, also compare against sibling quickstarts in the factory. Use the same scoring but compare techniques more deeply (grep source for shared patterns).

### Phase 7: Security Check

- Grep all files in the output directory for hardcoded secrets: `api_key\s*[:=]\s*["'][A-Za-z0-9]`, `password\s*[:=]\s*["'][A-Za-z0-9]`, `secret\s*[:=]\s*["'][A-Za-z0-9]`, `sk-[A-Za-z0-9]{20,}` (exclude .git/)
- Verify no .env files with real values
- Verify .gitignore covers .env, __pycache__, venv/
- If secrets are found in the source, REMOVE them from the output and add the variable to .env.example instead

### Phase 7: Static Validation

Before running, verify the output quickstart is internally consistent:

1. Do all internal links in the README resolve to real files in the output?
2. Does the Makefile reference paths that exist?
3. Does the validation_matrix stage_2_unit have real criteria (not empty)?
4. Does the claim_registry have entries for every number in the README?
5. Does the benchmark_rubric have thresholds matching the claims?

Fix any inconsistencies before proceeding.

### Phase 8: Run & Validate

Actually run the quickstart and verify it works. This is the difference between "the repo looks right" and "the repo actually works."

**Step 1: Install dependencies**

If the quickstart has a `pyproject.toml` or `requirements.txt`:

```bash
cd /tmp/quickstart-intake-<repo-name>
python3 -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]" 2>/dev/null || pip install -r requirements.txt 2>/dev/null || pip install -r src/requirements.txt 2>/dev/null
pip install pytest pyyaml 2>/dev/null
```

**Step 2: Run publication tests**

```bash
python -m pytest tests/publication/ -v --tb=short 2>&1
```

Report pass/fail. If tests fail, fix the README or test_readme.py to make them pass — the output must be green.

**Step 3: Run contract tests**

If `tests/contracts/` has test files:

```bash
python -m pytest tests/contracts/ -v --tb=short 2>&1
```

Report pass/fail. If OpenAPI specs don't parse, fix them.

**Step 4: Run unit tests**

If `tests/unit/` has test files:

```bash
python -m pytest tests/unit/ -v --tb=short 2>&1
```

Report pass/fail. Note: unit tests may need environment variables (MODEL_ENDPOINT, DEMO_MODE). Try with `DEMO_MODE=true` if available.

**Step 5: Helm template render** (if chart/ exists)

```bash
helm template test chart/ --values chart/values.yaml 2>&1
```

Report pass/fail. Template must render without errors.

**Step 6: Container build** (if Containerfile/Dockerfile exists)

```bash
podman build -t quickstart-test:latest -f src/Containerfile src/ 2>&1
```

Or if Containerfile is at root:

```bash
podman build -t quickstart-test:latest -f Containerfile . 2>&1
```

Report pass/fail. If the build fails, note the error but don't block — container builds may need registry access or specific base images.

**Step 7: Compose stack** (if docker-compose.yml exists)

```bash
podman compose up -d 2>&1
# Wait for health checks
sleep 10
# Check if services are running
podman compose ps 2>&1
# Hit health endpoint if the app exposes one
curl -sf http://localhost:8000/health 2>/dev/null && echo "HEALTH: OK" || echo "HEALTH: no response"
# Clean up
podman compose down -v 2>&1
```

Report pass/fail. Note: compose stacks may need model downloads or external services. If they fail due to missing models or network access, note it as EXPECTED rather than a failure.

**Step 8: Script execution** (for script-based quickstarts)

If the quickstart is script-based (like rhaiis-cpu-quickstart), run the main script in a dry-run or help mode if available:

```bash
bash -n start.sh 2>&1  # syntax check
```

Report pass/fail on syntax. Full execution may require hardware/registries — note as NEEDS HARDWARE.

**Reporting format for Phase 8:**

```
RUN & VALIDATE:
  [PASS] Publication tests (X/X passed)
  [PASS] Contract tests (X/X passed)
  [PASS] Unit tests (X/X passed)
  [PASS] Helm template renders
  [PASS] Gradio UI syntax valid
  [SKIP] Container build (needs registry access)
  [SKIP] Compose stack (needs model download)
```

If any test fails, fix it before reporting. The output quickstart must have green tests for the stages that can run without external dependencies (publication, contracts, unit tests with mocks/demo mode).

### Phase 9: Report

```
QUICKSTART INTAKE: <repo-name>
================================
Source: <source repo URL> (READ-ONLY — not modified)
Output: /tmp/quickstart-intake-<repo-name>
Type: <container / script / helm / notebook>

BUSINESS SOLUTION ..... [STRONG / MODERATE / WEAK / NONE]
  Problem: <one sentence>
  User: <role / industry>
  Industry: <from approved list>
  Teachable: <YES / PARTIAL / NO>

INTEL STORY ........... [STRONG / MODERATE / WEAK / NONE]
  Value prop: <one sentence>
  Branding: [OK / ISSUES]

GRADIO UI ............. [GENERATED / KEPT / ADAPTED]
  Tabs: <tab names>
  Backend var: <env var name>
  Endpoints wired: <count>

FILES CREATED:
  + src/ui.py (Gradio frontend, X tabs, X endpoint handlers)
  + tests/validation_matrix.yaml (6 stages, X criteria in stage_2_unit)
  + tests/claim_registry.yaml (X claims registered)
  + tests/benchmark_rubric.yaml (X benchmarks defined)
  + tests/publication/test_readme.py
  + tests/conftest.py
  + Makefile (X targets)
  + .github/workflows/ci.yaml
  + .env.example
  ...

FILES MODIFIED:
  ~ README.md (added Tags, restructured opening)
  ...

CLAIMS ................ [X registered, all unverified]
  "16 GB minimum" — README.md:52 — hardware requirement
  "~9 tok/s at 8 cores" — README.md:83 — Uncited, needs measurement
  ...

OVERLAP SCAN .......... [CLEAR / DUPLICATES / HIGH / MEDIUM]
  Org repos scanned: N
  [DUPLICATE] <name> — already in org
  [HIGH] <ours> ↔ <theirs> (shared: technique1, technique2)
  [MEDIUM] <ours> ↔ <theirs> (shared: keyword)

SECURITY .............. [CLEAN / ISSUES]
  ...

RUN & VALIDATE ........ [X/Y passed, Z skipped]
  [PASS] Publication tests (X/X)
  [PASS] Contract tests (X/X)
  [PASS] Unit tests (X/X)
  [PASS] Helm template renders
  [SKIP] Container build (needs registry)
  [SKIP] Compose stack (needs model)

REMAINING MANUAL STEP:
  Verify performance claims on target hardware and update
  tests/claim_registry.yaml: set verified: true + verified_date

READY TO PUSH: [YES / AFTER CLAIM VERIFICATION]
  Output directory: /tmp/quickstart-intake-<repo-name>
  To push: cd /tmp/quickstart-intake-<repo-name> && gh repo create ...
```

After the scorecard, give a 3-5 sentence summary: what the quickstart does, the business story, and confirmation that the output is ready to push (pending claim verification on hardware).

---

### Important Rules

- **Never modify the source repo.** Clone to `-source`, build the output in a separate directory.
- **Business solution is the gate check.** If there's no business problem, reframe it — don't just flag it.
- **Fill in everything.** No blank templates, no TODOs, no "customize per quickstart" placeholders. You read the source — use what you learned.
- **The output must be internally consistent.** test_readme.py must pass against the README. Makefile paths must resolve. Claims in the registry must match claims in the README.
- Be strict on security — remove real credentials, replace with .env.example references.
- Be pragmatic on structure — script-only repos are first-class. Don't force Helm charts where they don't belong.
- The only manual step left should be running benchmarks on real hardware to verify claims. Everything else is done.
