# ETHAND (2024)

The first draft of this project, written in 2024 under the name ETHAND (Ethical Training and Artificial Neural Development). Five Obsidian notes, no code. They are kept here unchanged as a record of where the design started; the current design is in [docs/proposal.md](../../proposal.md).

Files:

- `Readme.md`: the project pitch
- `Project.md`: planned package layout, setup steps, and a main configuration file
- `Requirements.md`: planned dependencies
- `Prompt.md`: the system prompt used with an AI assistant during development
- `Ethand.md`: the name's rationale

The notes contain generated filler (documentation links that never existed, a `logging>=0.5.0` dependency, a `sqlite3` pip line). They are left as written.

## What carried over

| ETHAND (2024) | Positronic Mind (2026) |
| --- | --- |
| `ethics/guardrails.py`, `filters.py`, `validator.py` | Conscience check plus Cedar veto; the model check may only tighten a rule verdict |
| `ethics/principles.py` with a priority schema | Values ledger of natural-language principles with provenance and owner confirmation |
| `reasoning/trace_db.py`, reasoning chain storage | Decision trace and why trace |
| `learning/knowledge_store.py`, NetworkX knowledge graph | Property graph with bi-temporal facts and derived-from edges |
| `learning/academic_sources.py`, source validator | Reader with source labels bound at write time |
| `continuous_learning.py` with incremental model updates | Learning lives in memory; one LoRA from a base checkpoint |
| "Continuous Development" | Consolidation cycle and curiosity module |

## What changed, and why

- ETHAND updated model weights on an hourly schedule. The 2026 research review found sequential weight updates forget (SEAL), so learning moved to memory.
- ETHAND read academic sources filtered by citation count and peer review. The new reader covers the open web with robots.txt, write-time source labels, and poisoning defenses, which the review showed matter more than source prestige.
- ETHAND had three fixed principles in YAML. The ledger now learns principles from the owner, each with evidence and a confirmation step.
- ETHAND had no refusal from its own values and no split between conscience and rules. That split is the central design choice now.
- ETHAND had no global workspace or faculty decomposition. Those came from the brain-mapping work in 2025 and 2026.
