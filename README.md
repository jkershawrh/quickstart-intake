# Quickstart Intake

Take an existing GitHub repo and turn it into a factory-standard Intel AI Quickstart.

## What it does

You point it at a repo. It:

1. **Clones and reads** the repo to understand what it does
2. **Runs a gap analysis** against the factory quickstart structure (tests, Makefile, validation matrix, claim registry, benchmark rubric, architecture diagrams, LICENSE, README sections)
3. **Scaffolds the missing pieces** — creates the factory test harness, claim registry, validation matrix, and Makefile around the existing code without touching it
4. **Flags README gaps** — reports what sections need to be added or restructured to match the publication standard, but doesn't rewrite without approval
5. **Audits claims** — finds performance numbers in the README and source, registers them as unverified claims that need measurement
6. **Checks Intel story and branding** — rates the Intel value prop and flags branding issues
7. **Reports a verdict** — READY to push, or AFTER FIXES with a specific action list

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
| `Makefile` | `make test-all` runs all 6 stages sequentially |
| `docs/images/` | Architecture diagrams |
| `LICENSE` | Apache 2.0 |

The intake skill wraps this scaffolding around repos that weren't built with the factory from the start.
