# deeds-ace-fixer

Agentic error resolution pipeline for large SvelteKit + TypeScript codebases. Three-phase orchestrator that moves from fast regex pattern fixes → structural import repair → AI-assisted semantic type fixes via Ollama.

Originally developed against the **deeds-web-app** legal AI platform (50,000+ TypeScript/Svelte errors after a Phase 99 corrupted migration). Designed to be reusable against any SvelteKit project.

---

## How It Works

```
Phase 1 — Automated (seconds, ~2K errors)
  Pattern-based regex fixes on raw source files:
  • Duplicate trailing quotes in imports
  • Invalid $state() placement inside callbacks/try-finally
  • Legacy Svelte 4 $: reactive declarations → $derived()
  • Component name casing normalization (dialog → Dialog)

Phase 2 — Structural (minutes, ~5K errors)
  Import path resolution:
  • Broken .js/.ts extension mismatches
  • Missing index barrel re-exports
  • Renamed/moved module paths

Phase 3 — AI-Assisted (hours, ~15-25K errors)
  Ollama (gemma3-legal or any local LLM) for semantic fixes:
  • Complex TypeScript type mismatches
  • XState v5 fromPromise generic signatures
  • Interface/type incompatibilities
  • Redis caching of fix patterns (avoid re-querying same error)
  • Qdrant vector search to find similar previously-fixed errors
```

---

## Quick Start

### Prerequisites

- Node.js 20+
- For Phase 3: Ollama running locally (`ollama serve`)
- For Phase 3 caching: Redis + Qdrant (optional but recommended)

### Run All Phases

```bash
cd scripts
node run-all-phases.mjs
```

### Run Individual Phases

```bash
node scripts/phase1-automated-fixes.mjs   # Fast regex, no AI needed
node scripts/phase2-import-fixes.mjs      # Import path repair
node scripts/phase3-ai-repair.mjs         # Ollama-backed semantic fixes
```

### Point at Your Project

By default scripts resolve `ROOT` two levels up from `scripts/`. To target a different project:

```bash
# In each script, update:
const ROOT = path.resolve('/path/to/your/sveltekit-project');
```

Or pass via env:

```bash
PROJECT_ROOT=/path/to/project node scripts/phase1-automated-fixes.mjs
```

---

## Recommendations

### When to Use Each Phase

| Situation | Start with |
|-----------|-----------|
| Fresh migration (Svelte 4 → 5) | Phase 1 — bulk rune pattern fixes |
| Renamed/moved files broke imports | Phase 2 — import path resolver |
| Type errors after library upgrade | Phase 3 — Ollama semantic repair |
| Post-corruption recovery (e.g. codemods gone wrong) | All 3 phases in sequence |

### Phase 3 Model Recommendations

| Model | Speed | Quality | Best for |
|-------|-------|---------|----------|
| `gemma3-legal:latest` | Medium | High (legal domain) | deeds-web-app specific |
| `gemma3:latest` | Medium | Good | General SvelteKit projects |
| `llama3.2:latest` | Fast | Good | Quick passes |
| `codellama:latest` | Slow | Best | Deep type inference errors |

Set via env: `OLLAMA_MODEL=codellama:latest node scripts/phase3-ai-repair.mjs`

### Workflow for Large Error Counts (10K+)

1. **Commit current state** — always have a rollback point
2. Run Phase 1 → verify error count drops → commit
3. Run Phase 2 → verify → commit  
4. Run Phase 3 in batches of 50-100 files
5. Run `npx svelte-check` after each batch
6. If error count *increases*, `git reset --hard HEAD` and adjust batch size

### Redis + Qdrant Acceleration (Phase 3)

Phase 3 caches fix patterns so the same error shape doesn't hit Ollama twice:

```bash
# Start services (if using deeds-web-app docker-compose)
docker compose up redis qdrant -d

# Or set endpoints explicitly
REDIS_URL=redis://localhost:6379 \
QDRANT_URL=http://localhost:6333 \
node scripts/phase3-ai-repair.mjs
```

Without Redis/Qdrant, Phase 3 still works — just slower (no fix reuse).

---

## Output Structure

```
errors/
  phase1-summary.json    ← files scanned, patterns matched, errors estimated fixed
  phase2-summary.json    ← imports resolved / failed
logs/
  phase1-{timestamp}.log
  phase3-{timestamp}.log
reports/
  phase1-report.json
  phase2-report.json
fixed/
  phase1/                ← snapshots of files modified by phase 1
  phase2/                ← snapshots of files modified by phase 2
```

---

## Origin

Built during cleanup of **deeds-web-app** (Legal AI case management platform). After a corrupted AST migration tool generated 83 `.svelte` files with broken syntax across 50,000+ errors, this pipeline recovered the codebase to `svelte-check: 0 errors` without manual file-by-file editing.

Related repo: [evidence_microservice](https://github.com/semaj90/evidence_microservice) — the evidence processing microservice extracted from the same project.