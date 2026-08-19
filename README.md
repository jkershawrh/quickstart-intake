# Quickstart Intake

Take an existing GitHub repo and turn it into a factory-standard Intel AI Quickstart. Business solution first, tech second.

## What it does

You point `/intake` at a repo. It:

1. **Reads the repo** to understand what it does
2. **Assesses the business solution** (GATE CHECK) — does this solve a real industry problem? Is the business value clear before the tech details? Can someone learn HOW to build it? If there's no business story, it flags that before anything else
3. **Runs a gap analysis** against the full factory structure (tests, Makefile, validation matrix, claim registry, benchmark rubric, CI workflow, architecture diagrams, .env.example, LICENSE)
4. **Scaffolds the missing pieces** — creates the factory test harness and build tooling around the existing code without touching it
5. **Assesses the README** — reports what sections need to be added to meet publication standards (business problem first)
6. **Audits claims** — finds performance numbers, registers them as unverified claims
7. **Checks Intel story and branding** — "Powered by Intel" OK, "Gaudi" in branding not OK
8. **Checks security** — no hardcoded secrets, .env coverage
9. **Reports a verdict** — READY, NEEDS WORK, or NOT READY with specific action items

## The gate check

The first thing the skill evaluates is the business solution — not the code quality. Kelkhund's approval criteria for the `rh-ai-quickstart` org: business solution comes first, technical "how it's done" comes second. A technically perfect quickstart with no business story won't get published.

It also assesses teachability: can a user learn HOW to build this, not just run it? This matters for showroom lab conversion downstream.

## What it doesn't do

- It doesn't rewrite your code or change your repo's architecture
- It doesn't replace your README — it tells you what's missing
- It doesn't require every repo to have a Helm chart — script-only quickstarts are first-class

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

Every quickstart produced by the factory ships with:

| Component | Purpose |
|-----------|---------|
| `tests/validation_matrix.yaml` | 6-stage CDD->TDD->EDD gate definitions |
| `tests/claim_registry.yaml` | Every factual assertion tracked with provenance |
| `tests/benchmark_rubric.yaml` | Technique-specific performance thresholds |
| `tests/publication/test_readme.py` | Automated README structure validation |
| `tests/conftest.py` | Shared test fixtures |
| `Makefile` | `make test-all` runs all 6 stages sequentially |
| `.github/workflows/ci.yaml` | CI runs contracts + publication checks on push |
| `.env.example` | Environment variable template with placeholders |
| `docs/images/` | Architecture diagrams |
| `LICENSE` | Apache 2.0 |

## Pipeline position

**repo** --`/intake`--> **quickstart** --`/onboard`--> **showroom lab**
