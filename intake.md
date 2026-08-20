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

FILES CREATED:
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
