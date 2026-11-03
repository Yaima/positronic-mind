# Positronic Mind

A personal AI agent that learns one person's values from interaction and reading, holds them in a ledger with provenance, and checks every action against them before acting. Built as a modular cognitive architecture around a global workspace, on open-weight models.

The name is an homage to Asimov's positronic brain and to Andrew Martin of *The Bicentennial Man*. It names no mechanism. The reference instance is called Andrew.

Status: Phase 0, not usable. Nothing runs yet.

## What it is

- A global workspace: modules post messages with a salience score, the winner is broadcast to every module, every message is recorded.
- Memory with provenance: episodes carry a source label bound at write time; semantic facts live in a property graph with bi-temporal timestamps and derived-from edges; contradictions are versioned, never overwritten.
- A values ledger: natural-language principles the owner confirms, each linked to the episodes that produced it, backed by structured rules where a principle can be expressed as one.
- Two sources of refusal, in order: the engine's own conscience (an LLM check against the principles), then a deterministic veto (Cedar rules through AgentTrust ID). The model check can only make a decision stricter.
- A decision trace: every action records its inputs, the conscience verdict, the matched rule and version, and the decision.

## Documents

- [docs/proposal.md](docs/proposal.md): the design, faculty map, build plan, evaluation, risks.
- [docs/research-review.md](docs/research-review.md): survey of similar work, evidence per architectural choice, constraints, with sources.
- [docs/decisions.md](docs/decisions.md): dated decision log.
- [docs/experiments/](docs/experiments/): one folder per decision experiment, each with hypothesis, metric, and result.
- [docs/history/ethand-2024/](docs/history/ethand-2024/): the 2024 first draft of this project, kept as written.

## Build plan

| Phase | Build |
| --- | --- |
| 0 | OLMo-3-7B base under llama.cpp; global workspace with LLM core, working memory, execution; 50-task baseline suite |
| 1 | Decision trace and veto (Cedar rules, conscience check that can only tighten, sandbox) |
| 2 | Episode store, property graph, dedup-only nightly consolidation |
| 3 | Values ledger with provenance; principle-to-rule compilation; owner approval |
| 4 | One LoRA from the base checkpoint for conscience refusal |
| 5 | Web reader honoring robots.txt; curiosity by prediction error |
| 6 | Remaining modules, each kept only behind an ablation against a single-loop arm |

Each phase has a decision experiment in the proposal.

## Public engine, private instance

The engine is public. An instance of it (`values/values.md`, episodes, the graph, reading notes, the trace) is one person's data and stays out of git. `.gitignore` covers those paths; `values/values.example.md` is a fictional sample.

## Development

This repo uses the [SDLC pipeline](https://github.com/Yaima/sdlc-pipeline). Read `AGENTS.md` and `docs/pipeline/PROCESS.md` before any pipeline work. Commits, pushes, and merges are human actions.

```
python -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"
python -m pytest
```

## License

Apache 2.0. See [LICENSE](LICENSE).
