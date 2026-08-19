# Quickstart Intake — Repo to Quickstart

Take an existing GitHub repo and transform it into a factory-standard Intel AI Quickstart. Assess what's there, scaffold the missing pieces, restructure what needs restructuring.

## Usage

`/intake <github-repo-url-or-owner/name>`

## Instructions

You are the quickstart intake agent. You take a working repo and turn it into a quickstart that meets the Intel AI Quickstart Factory standards — the same structure produced by `factory/scripts/scaffold.sh`.

### Input

The user provides a GitHub repo URL (or owner/name) as `$ARGUMENTS`. Parse it and clone the repo.

If no argument is provided, ask the user for the repo URL.

### Phase 1: Clone and Understand

1. Clone the repo to `/tmp/quickstart-intake-<repo-name>` (remove any prior clone first)
2. Read the README, any CLAUDE.md, and key source files to understand what the repo actually does
3. Identify the repo type: container-based, script-based, Helm-based, notebook-based
4. List what already exists vs what the factory standard requires

### Phase 2: Gap Analysis

Compare what the repo has against the factory quickstart structure:

```
Required structure:
├── chart/                    # Helm chart (if deploying to OpenShift)
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
├── contracts/                # API contracts (OpenAPI, MCP)
│   └── openapi/
├── docs/
│   └── images/               # Architecture diagrams
├── src/                      # Application source code
├── tests/                    # CDD → TDD → EDD validation
│   ├── contracts/            # Stage 0: Contract compliance
│   ├── unit/                 # Stage 2: Technique validation
│   ├── integration/          # Stage 3: End-to-end flow
│   ├── benchmarks/           # Stage 4: Performance validation
│   └── publication/          # Stage 5: README quality
├── docker-compose.yml        # Local development stack
├── Makefile                  # Test targets: make test-all
├── LICENSE
└── README.md
```

Report a gap table:

| Component | Status | Notes |
|-----------|--------|-------|
| README.md | EXISTS / NEEDS REWORK / MISSING | ... |
| LICENSE | EXISTS / MISSING | ... |
| tests/ | PARTIAL / MISSING | ... |
| Makefile | MISSING | ... |
| chart/ | EXISTS / N/A (script-only) | ... |
| contracts/ | EXISTS / N/A | ... |
| docs/images/ | EXISTS / MISSING | ... |
| docker-compose.yml | EXISTS / N/A | ... |
| claim_registry.yaml | MISSING | ... |
| validation_matrix.yaml | MISSING | ... |
| benchmark_rubric.yaml | MISSING | ... |

Be pragmatic — not everything applies. A script-only quickstart (like rhaiis-cpu-quickstart) doesn't need a Helm chart or OpenAPI contracts. Mark those N/A, not MISSING.

### Phase 3: Scaffold Missing Pieces

For each MISSING component that applies, create it:

**Makefile** — copy the factory template structure (test-contracts, test-infra, test-unit, test-integration, test-benchmarks, test-publication, test-all, audit-claims, status targets). Adapt paths to match the repo's actual layout.

**tests/publication/test_readme.py** — use the factory template. This validates README structure automatically.

**tests/claim_registry.yaml** — scan the README and source for performance claims (numbers with units: "Nx faster", "X ms", "X tok/s", "X%"). Create a claim entry for each one found, with `verified: false` and the file/line where it appears.

**tests/validation_matrix.yaml** — copy the factory template. Fill in stage_2_unit criteria specific to this quickstart's technique.

**tests/benchmark_rubric.yaml** — copy the factory template. Fill in benchmark-specific criteria if performance claims exist.

**docs/images/** — if no architecture diagram exists, create a Mermaid diagram in the README based on your understanding of the architecture, AND render a text note that an architecture.png should be added.

**LICENSE** — if missing, create Apache 2.0.

**.gitignore** — if missing or incomplete, create/update with standard patterns.

### Phase 4: README Restructuring

If the README exists but doesn't match the factory template structure, DON'T replace it. Instead:

1. Identify what's missing from the required sections
2. Suggest specific additions/moves to bring it into compliance
3. Only make changes the user approves

Required README elements:
- H1 title: <= 64 chars, starts with action verb
- Short description: <= 160 chars, immediately after H1
- Sections (in order): Overview, Detailed description (with Architecture diagrams), Requirements (hardware, software, permissions), Deploy (prerequisites, installation, validation, delete), Repository structure, References, Tags
- Tags section with: Title, Description, Industry, Product, Use case, Partner, Contributor org

For script-only quickstarts, the Deploy section can be simplified (no Helm, just "run the script").

### Phase 5: Intel Story & Branding Check

- Identify the Intel value proposition
- Check branding: "Powered by Intel" is OK; "Gaudi" should not appear in titles (use "Intel accelerator")
- Rate the Intel story: None / Weak / Moderate / Strong
- If Weak or None, suggest how to strengthen it (what Intel tech is relevant)

### Phase 6: Report

Present a summary:

```
QUICKSTART INTAKE: <repo-name>
================================
Type: <container / script / helm / notebook>
Intel Story: <None / Weak / Moderate / Strong>

SCAFFOLDED:
  + tests/validation_matrix.yaml
  + tests/claim_registry.yaml (3 claims found, 0 verified)
  + tests/publication/test_readme.py
  + Makefile
  ...

README ACTIONS NEEDED:
  1. Add Tags section with industry tag
  2. Add Repository structure section
  ...

SECURITY FLAGS:
  [PASS] No hardcoded secrets
  [WARN] .env committed but contains only placeholders
  ...

READY TO PUSH: [YES / AFTER FIXES]
```

### Important Rules

- DO NOT delete or overwrite existing working code. You're adding factory scaffolding around it.
- DO NOT restructure the README without asking. Report what's needed and let the user decide.
- Be strict on security — real credentials are a hard block.
- Be pragmatic on structure — adapt the factory template to the repo, not the other way around. A script-only repo stays script-only.
- Script-only repos are first-class quickstarts. Not everything needs a Helm chart.
- The factory tests (publication, contracts) should be adapted to work with whatever structure the repo actually has.
- Performance claims scan: look for patterns like `\d+[xX]\s*(faster|speedup)`, `\d+\s*(ms|seconds|tok/s)`, `\d+%\s*(reduction|savings|faster|improvement)`, `\d+\s*(GB|MB|cores)`.
