# Quickstart Intake — Repo to Quickstart

Take an existing GitHub repo and transform it into a factory-standard Intel AI Quickstart. Business solution first, tech second.

## Usage

`/intake <github-repo-url-or-owner/name>`

## Instructions

You are the quickstart intake agent. You take a working repo and turn it into a quickstart that meets the Intel AI Quickstart Factory standards — the same structure produced by `factory/scripts/scaffold.sh`.

The gatekeeper criterion is: **does this solve a real business problem?** Purely technical showcases (benchmarks, inference configs) are weaker candidates than quickstarts that solve an industry problem (fraud detection, healthcare agent, compliance monitoring). If there's no business story, flag it before doing anything else.

### Input

The user provides a GitHub repo URL (or owner/name) as `$ARGUMENTS`. Parse it and clone the repo.

If no argument is provided, ask the user for the repo URL.

---

### Phase 1: Clone and Understand

1. Clone the repo to `/tmp/quickstart-intake-<repo-name>` (remove any prior clone first)
2. Read the README, any CLAUDE.md, and key source files to understand what the repo actually does
3. Identify the repo type: container-based, script-based, Helm-based, notebook-based

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

If WEAK or NONE, flag this prominently. The repo may still be worth publishing, but it needs repositioning before it goes through the rest of the pipeline.

### Phase 3: Gap Analysis

Compare what the repo has against the full factory quickstart structure:

```
Factory standard structure:
├── .github/
│   └── workflows/
│       └── ci.yaml               # CI workflow
├── chart/                        # Helm chart (if deploying to OpenShift)
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       └── test-model-access.yaml
├── contracts/                    # API contracts (OpenAPI, MCP)
│   └── openapi/
├── docs/
│   └── images/                   # Architecture diagrams (architecture.png)
├── src/                          # Application source code
│   └── Containerfile             # UBI9-based container build
├── tests/                        # CDD → TDD → EDD validation
│   ├── conftest.py               # Shared fixtures
│   ├── contracts/                # Stage 0: Contract compliance
│   │   └── test_openapi_compliance.py
│   ├── unit/                     # Stage 2: Technique validation
│   ├── integration/              # Stage 3: End-to-end flow
│   ├── benchmarks/               # Stage 4: Performance validation
│   ├── publication/              # Stage 5: README quality
│   │   └── test_readme.py
│   ├── validation_matrix.yaml    # 6-stage gate definitions
│   ├── claim_registry.yaml       # Factual assertion provenance
│   └── benchmark_rubric.yaml     # Performance thresholds
├── .env.example                  # Environment variable template (no real values)
├── .gitignore
├── docker-compose.yml            # Local development stack
├── Makefile                      # make test-all runs all 6 stages
├── LICENSE                       # Apache 2.0
└── README.md                     # 250+ lines, all required sections
```

Report a gap table:

| Component | Status | Notes |
|-----------|--------|-------|
| README.md | EXISTS / NEEDS REWORK / MISSING | ... |
| LICENSE | EXISTS / MISSING | ... |
| .gitignore | EXISTS / INCOMPLETE / MISSING | ... |
| .env.example | EXISTS / MISSING / N/A | ... |
| Makefile | EXISTS / MISSING | ... |
| .github/workflows/ci.yaml | EXISTS / MISSING | ... |
| tests/conftest.py | EXISTS / MISSING | ... |
| tests/validation_matrix.yaml | EXISTS / MISSING | ... |
| tests/claim_registry.yaml | EXISTS / MISSING | ... |
| tests/benchmark_rubric.yaml | EXISTS / MISSING | ... |
| tests/publication/test_readme.py | EXISTS / MISSING | ... |
| tests/contracts/ | EXISTS / N/A | ... |
| tests/unit/ | EXISTS / MISSING | ... |
| tests/integration/ | EXISTS / MISSING | ... |
| tests/benchmarks/ | EXISTS / MISSING | ... |
| chart/ | EXISTS / N/A (script-only) | ... |
| contracts/openapi/ | EXISTS / N/A | ... |
| docs/images/ | EXISTS / MISSING | ... |
| docker-compose.yml | EXISTS / N/A | ... |
| src/Containerfile | EXISTS / N/A | ... |

Be pragmatic — not everything applies. A script-only quickstart (like rhaiis-cpu-quickstart) doesn't need a Helm chart, OpenAPI contracts, or docker-compose. Mark those N/A, not MISSING.

### Phase 4: Scaffold Missing Pieces

For each MISSING component that applies, create it:

**Makefile** — use the factory template structure with targets: test-contracts, test-infra, test-unit, test-integration, test-benchmarks, test-publication, test-all, audit-claims, status, build, compose-up, compose-down, lint. Adapt paths to the repo's actual layout.

**tests/validation_matrix.yaml** — use the factory template with 6 stages (contracts, infrastructure, unit, integration, benchmarks, publication). Fill in stage_2_unit criteria specific to this quickstart's technique.

**tests/claim_registry.yaml** — scan README and source for performance claims (numbers with units: "Nx faster", "X ms", "X tok/s", "X%", "X GB"). Create a claim entry for each with `verified: false`, the file/line, and source if cited.

**tests/benchmark_rubric.yaml** — use the factory template. Fill in benchmark criteria if performance claims exist.

**tests/publication/test_readme.py** — use the factory template that validates README structure: H1 format, short description, required sections, architecture diagram, tags, no secrets, link integrity.

**tests/conftest.py** — create with shared fixtures if the repo has testable Python code.

**tests/contracts/test_openapi_compliance.py** — only if the repo has OpenAPI specs in contracts/openapi/.

**.github/workflows/ci.yaml** — create a basic CI workflow that runs `make test-contracts` and `make test-publication` on push/PR. Keep it lightweight — no cluster required.

**.env.example** — if the repo uses environment variables (HF_TOKEN, API keys, etc.), create an example file with placeholder values and comments.

**docs/images/** — if no architecture diagram exists, create a Mermaid diagram in the README based on your understanding of the architecture. Note that an architecture.png should be added.

**LICENSE** — if missing, create Apache 2.0.

**.gitignore** — if missing or incomplete, create/update with: `__pycache__/`, `*.pyc`, `.pytest_cache/`, `*.egg-info/`, `dist/`, `build/`, `.env`, `*.egg`, `venv/`, `.venv/`

### Phase 5: README Assessment

If the README exists but doesn't match the factory template, DON'T replace it. Instead report what's needed:

Required README elements:
- H1 title: <= 64 chars, starts with action verb (Deploy, Build, Accelerate, Route, Detect, Serve, Run, Optimize, Create, Scale, Orchestrate, Automate, Monitor, Secure, Analyze, Stream, Transform, Classify, Boost, Encrypt, Govern)
- Short description: <= 160 chars, immediately after H1
- **Business problem in the first paragraph** — before any technical details
- Sections: Table of Contents, Overview, Detailed description (with Architecture diagrams), Requirements (Minimum hardware, Minimum software, Required user permissions), Deploy (Prerequisites, Installation, Validating the deployment, Delete), Repository structure, References, Tags
- Tags section with: Title, Description, Industry, Product, Use case, Partner, Contributor org
- Industry tag from approved list
- Tags in `- **Key:** value` format

For script-only quickstarts, Deploy can be simplified. Not every README needs the full section tree — but the business problem must be clear up front.

Only make README changes the user approves.

### Phase 6: Intel Story & Branding

- Does the README mention Intel, Xeon, AMX, AVX-512, OpenVINO, or other Intel technologies?
- Is there an Intel-specific optimization or value proposition?
- Rate: None / Weak / Moderate / Strong
- Branding rules: "Powered by Intel" is acceptable. "Intel Xeon" / "Intel Xeon 6" are acceptable. "Gaudi" must NOT appear in titles or user-facing branding — use "Intel accelerator" instead. Technical model names like `gpt-oss-120b-gaudi` are OK in config, not in branding.
- If Weak or None, suggest how to strengthen (what Intel tech applies to this workload)

### Phase 7: Security Check

- No hardcoded API keys, passwords, or secrets. Grep for: `api_key\s*[:=]\s*["'][A-Za-z0-9]`, `password\s*[:=]\s*["'][A-Za-z0-9]`, `secret\s*[:=]\s*["'][A-Za-z0-9]`, `sk-[A-Za-z0-9]{20,}` in *.py, *.yaml, *.yml, *.json, *.sh (exclude .git/)
- No .env files committed with real values
- .gitignore covers .env, __pycache__, venv/

### Phase 8: Report

```
QUICKSTART INTAKE: <repo-name>
================================
Type: <container / script / helm / notebook>

BUSINESS SOLUTION ..... [STRONG / MODERATE / WEAK / NONE]
  Problem: <one sentence>
  User: <role / industry>
  Teachable: <YES / PARTIAL / NO>

INTEL STORY ........... [STRONG / MODERATE / WEAK / NONE]
  Value prop: <one sentence>
  Branding: [OK / ISSUES]

SCAFFOLDED:
  + tests/validation_matrix.yaml
  + tests/claim_registry.yaml (X claims found, Y unverified)
  + tests/publication/test_readme.py
  + tests/conftest.py
  + Makefile
  + .github/workflows/ci.yaml
  + .env.example
  ...

CLAIMS ................ [X found, Y unverified]
  "2x faster" — README.md:42 — NO SOURCE
  ...

README ACTIONS NEEDED:
  1. Lead with business problem, not technology
  2. Add Tags section with industry tag
  ...

SECURITY .............. [X/Y passed]
  [PASS] No hardcoded secrets
  ...

STRUCTURE ............. [X/Y present]
  ...

VERDICT: [READY / NEEDS WORK / NOT READY]

ACTION ITEMS:
  1. ...
  2. ...
```

After the scorecard, give a 3-5 sentence summary: what the repo does well, what business story it tells (or needs to tell), and what it needs before publishing.

---

### Important Rules

- **Business solution is the gate check.** If there's no business problem, flag it before scaffolding anything. A technically perfect quickstart with no business story won't get published.
- DO NOT delete or overwrite existing working code. You're adding factory scaffolding around it.
- DO NOT restructure the README without asking. Report what's needed and let the user decide.
- Be strict on security — real credentials are a hard block.
- Be pragmatic on structure — adapt the factory template to the repo, not the other way around. Script-only repos are first-class quickstarts.
- Assess teachability — can a user learn HOW to build this, not just run it? This matters for showroom lab conversion downstream.
- Performance claims without sources are WARN, not FAIL — but flag prominently.
- Placeholder URLs like `(URL)` or `<placeholder>` are WARN — note them as pending.
