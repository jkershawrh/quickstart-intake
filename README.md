# Quickstart Intake

Take an existing GitHub repo and produce a complete, ready-to-push Intel AI Quickstart. Business solution first, tech second.

## What it does

You point `/intake` at a repo. It:

1. **Clones the source repo read-only** — never modifies it
2. **Assesses the business solution** (GATE CHECK) — does this solve a real industry problem? Is the business value clear before the tech details? Can someone learn HOW to build it? If the business story is weak, it reframes it
3. **Builds a complete quickstart** in a separate output directory — copies the source, then fills in every missing factory component with real content (not blank templates)
4. **Rewrites the README** to lead with the business problem, adds all required sections (Table of Contents, Overview, Architecture, Requirements, Deploy, Repository structure, References, Tags), and ensures it passes `test_readme.py`
5. **Registers all claims** — scans README and source for performance numbers, creates a claim registry entry for each with file/line and verification status
6. **Fills in technique-specific test criteria** — reads the source code and writes real `stage_2_unit` criteria in the validation matrix, not placeholders
7. **Generates a Gradio frontend** (`src/ui.py`) — inspects the repo's API surface (OpenAPI specs, FastAPI routes, README curl examples), then builds a tabbed Gradio UI with pre-filled example inputs, `httpx` backend helpers, and an AI disclaimer. Matches the `gr.Blocks` pattern used across all 35 existing quickstarts. Keeps existing Gradio UIs if present
8. **Checks Intel story and branding** — rates the value prop, suggests strengthening if weak, enforces "Powered by Intel" rules
9. **Checks security** — scans for hardcoded secrets, removes any found, adds variables to `.env.example`
10. **Scans for overlap** — compares against all repos in the `rh-ai-quickstart` org and local sibling quickstarts to flag duplicates or high-overlap entries before you publish
11. **Self-validates** — verifies the output is internally consistent (README passes test_readme.py, Makefile paths resolve, claims match registry, Gradio UI syntax parses)
12. **Runs the quickstart** — installs deps, runs publication/contract/unit tests, renders Helm templates, attempts container builds and compose stacks
13. **Reports a scorecard** — business solution rating, Intel story, Gradio UI status, files created/modified, claims registered, overlap scan, security, run results, verdict

## The gate check

The first thing the skill evaluates is the business solution — not the code quality. Kelkhund's approval criteria for the `rh-ai-quickstart` org: business solution comes first, technical "how it's done" comes second. A technically perfect quickstart with no business story won't get published.

It also assesses teachability: can a user learn HOW to build this, not just run it? This matters for showroom lab conversion downstream.

## What it produces

A complete output directory at `/tmp/quickstart-intake-<repo-name>` containing:

- All original source code (untouched)
- Factory-standard README with business-first framing and all required sections
- All factory scaffolding filled in with real content
- The only remaining manual step: run benchmarks on target hardware to verify claims

## What it doesn't do

- It doesn't modify the source repo — output goes to a separate directory
- It doesn't rewrite your application code or change your architecture
- It doesn't require every repo to have a Helm chart — script-only quickstarts are first-class
- It doesn't verify performance claims — that requires real hardware

## Usage as a Claude Code skill

Copy `intake.md` to your project's `.claude/commands/` directory:

```bash
mkdir -p .claude/commands
cp intake.md .claude/commands/intake.md
```

Then invoke:

```
/intake https://github.com/owner/repo
```

## The factory standard

Every quickstart produced by the intake ships with:

| Component | Purpose |
|-----------|---------|
| `tests/validation_matrix.yaml` | 6-stage CDD->TDD->EDD gate definitions with real criteria |
| `tests/claim_registry.yaml` | Every factual assertion tracked with provenance |
| `tests/benchmark_rubric.yaml` | Technique-specific performance thresholds |
| `tests/publication/test_readme.py` | Automated README structure validation |
| `tests/conftest.py` | Shared test fixtures |
| `Makefile` | `make test-all` runs all 6 stages sequentially |
| `.github/workflows/ci.yaml` | CI runs contracts + publication checks on push |
| `src/ui.py` | Gradio frontend — tabbed UI wired to backend API endpoints |
| `.env.example` | Environment variable template with placeholders |
| `docs/images/` | Architecture diagrams |
| `LICENSE` | Apache 2.0 |
| `README.md` | Business-first, all required sections, passes test_readme.py |

## Pipeline position

**repo** --`/intake`--> **quickstart** --`/onboard`--> **showroom lab**
