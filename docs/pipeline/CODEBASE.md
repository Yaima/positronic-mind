
# Codebase map (generated 2026-10-29, rerun 2026-11-02, commit none — `git log` exits 128, "branch 'main' does not have any commits yet")

Phase 0 skeleton. Nine Python files totalling 13 lines of code: one version constant, seven
package docstrings, one test. Everything else on disk is the SDLC pipeline (93 `.md` files,
11 `.sh` hooks) and the design documents. `docs/proposal.md` is the design intent; this file
records only what exists.

## Commands (from `docs/pipeline/pipeline.json`, verified against `pyproject.toml`)

Run from the repo root with the venv active (`source .venv/bin/activate`). Nothing else needs
to be running — no DB, no services.

| purpose | command | verified |
|---|---|---|
| test | `python -m pytest -q` | exit 0, 1 passed |
| one test file | `python -m pytest -q {file}` | exit 0 on `tests/test_package.py` |
| lint | `python -m ruff check src tests` | exit 0, all checks passed |
| typecheck | `python -m mypy` | exit 0, 8 source files |
| format | `python -m ruff format src tests` | not run (writes files) |
| security | `python -m pip_audit` | exit 0, no known vulnerabilities |
| coverage | none (`coverage_cmd: ""`) | — |

Setup: `python -m venv .venv && source .venv/bin/activate && pip install -e ".[dev]"`
(`pyproject.toml:16` dev extra = pytest>=8, ruff>=0.6, mypy>=1.10, pip-audit>=2.7).

## Layout

```
src/positronic_mind/   7 subpackages, docstring-only (13 LOC total)
tests/                 1 file, flat
docs/
  proposal.md          design (341+ lines, 17 sections)
  research-review.md   evidence base
  decisions.md         decision log
  pipeline/            SDLC machinery: rules, roles, stages, templates, enforcement
  features/            per-feature dirs (empty: .gitkeep only)
  history/ethand-2024/ prior-art notes from an earlier project
  start/, experiments/ onboarding and experiment docs
eval/suite/            50-task baseline (empty: .gitkeep only)
eval/experiments/      experiment runs (empty: .gitkeep only)
values/values.example.md  fictional sample ledger
scripts/               empty (.gitkeep only)
.claude/ .agents/      12 stage skills, 9 role agents, 8 hooks
```

## Layers (dir → purpose → example file)

One subpackage per architectural layer from `docs/proposal.md`. All seven are
docstring-only placeholders — no classes, functions, or types exist yet.

| dir | purpose | file |
|---|---|---|
| `workspace/` | global workspace, salience, controller (≤3 modules/tick) | `src/positronic_mind/workspace/__init__.py:1` |
| `modules/` | faculty modules | `src/positronic_mind/modules/__init__.py:1` |
| `conscience/` | principle check (LLM, tighten-only) | `src/positronic_mind/conscience/__init__.py:1` |
| `veto/` | Cedar rules, AgentTrust ID bridge | `src/positronic_mind/veto/__init__.py:1` |
| `trace/` | decision event schema | `src/positronic_mind/trace/__init__.py:1` |
| `memory/` | episodes, graph, consolidation | `src/positronic_mind/memory/__init__.py:1` |
| `reader/` | web reading | `src/positronic_mind/reader/__init__.py:1` |

Each body is exactly `"""<name> package. See docs/proposal.md."""`.

## Entry points

None. No `[project.scripts]` in `pyproject.toml`, no `__main__.py`, no `if __name__`
anywhere under `src/`. The only importable symbol is
`positronic_mind.__version__ = "0.0.1"` (`src/positronic_mind/__init__.py:3`).

## Tests (framework, layout, run one file, examples)

- Framework: pytest (`pyproject.toml:16`), `testpaths = ["tests"]` (`pyproject.toml:31-32`).
- Layout: flat. `tests/` holds one file and no subdirectories — it does not yet mirror `src/`.
- Naming: `test_*.py`, bare `def test_*() -> None` functions. Return annotations are required
  (mypy strict, `pyproject.toml:27`).
- Fixtures: none. No `conftest.py` exists anywhere in the repo.
- Run one file: `python -m pytest -q tests/test_package.py`.
- Examples: `tests/test_package.py:4` (the only test). No second test file exists to cite.
- `test_globs` in `pipeline.json:10-15` still lists four layouts; only `**/test_*.py` and
  `**/tests/**` match anything on disk.

## Conventions (pattern → example file:line)

| pattern | example |
|---|---|
| Module docstring on every package | `src/positronic_mind/veto/__init__.py:1` |
| Explicit return annotations (mypy strict) | `tests/test_package.py:4` |
| Line length 100, target py311 | `pyproject.toml:22-23` |
| src-layout, setuptools find | `pyproject.toml:18-19` |
| Instance data gitignored, never committed | `.gitignore:2-11` (values.md, trace/, memory/) |
| Test files read-only for implementer | `.claude/hooks/guard-paths.sh:3` (`tests-readonly`) |
| Merge stays human; destructive cmds blocked | `.claude/hooks/guard-git.sh:2` |
| Agents must emit `## Evidence` + exit codes | `.claude/hooks/evidence-gate.sh:2-5` |
| Reviewer cannot read implementer fix notes | `.claude/hooks/guard-reviewer-inputs.sh:2` |
| CI gate keys on `Feature: <slug>` in PR body | `.github/workflows/sdlc-check.yml:21-23` |

Error handling, logging, DI, validation, and auth patterns: **none exist yet** — there is no
code to establish them. The first feature to add real code sets them.

## Data model (schema/migrations location)

No schema files, no migrations directory, no ORM. `dependencies = []`
(`pyproject.toml:13`). Nothing is persisted.

Designed but unbuilt, in `docs/proposal.md`:
- Semantic memory: typed property graph + vector store (`docs/proposal.md:97`). Entity types
  from schema.org (Person, Organization, Place, Event, CreativeWork) plus custom Value,
  Policy, Preference, Commitment, Episode. Provenance as derived-from edges; contradictions
  versioned, never overwritten.
- Workspace message fields: `source`, `content`, `confidence`, `salience`, `timestamp`
  (`docs/proposal.md:95`).
- Value representation: principle (source of truth) + zero or more derived rules
  (`docs/proposal.md:108`). The LLM check may only make a decision stricter
  (`docs/proposal.md:127`).
- `.gitignore:5-11` reserves `memory/episodes/`, `memory/graph/`, `trace/`, `*.db`,
  `*.sqlite` for the store when it lands.

## External boundaries

None in code. Zero runtime dependencies; no HTTP client, queue, or SDK imported anywhere
under `src/`.

Boundaries the pipeline itself crosses:
- GitHub: `origin → https://github.com/Yaima/positronic-mind.git`, PRs via `gh`
  (`pipeline.json:57`), gate in `.github/workflows/sdlc-check.yml`.
- PyPI: `pip-audit` queries it (`pipeline.json:7`).

Planned (`docs/proposal.md:293`): local open-weight models via llama.cpp or Ollama, and a
robots.txt-honoring web reader. Neither exists.

## Known gaps (things looked for and not found)

Rerun 2026-11-02. Resolved since the 2026-10-29 run and dropped from this list: the
`pip_audit` failure (fixed in both places — see below), the `board/serve.py` reference in
`subagent-log.sh`, and three Repo-facts lines in the project instructions (`tests/` flat,
`eval/suite/` and `eval/experiments/` reserved-but-empty) which now match disk.

**Verified failures**

1. **Python version split — open.** Tests execute on **3.14.5** (`.venv/bin/python`; the only
   interpreter installed, `/opt/homebrew/bin/python3.14`), while ruff `target-version = "py311"`
   (`pyproject.toml:23`), mypy `python_version = "3.11"` (`pyproject.toml:26`), and CI
   `python-version: "3.11"` (`.github/workflows/sdlc-check.yml:14`) all pin 3.11. Linting
   against the 3.11 floor is correct for a library declaring `requires-python = ">=3.11"`
   (`pyproject.toml:10`) — it catches 3.12+-only syntax a 3.11 user would hit. The real gap is
   coverage: CI exercises only 3.11 and the developer only 3.14, so neither environment tests
   the other. **Decided 2026-11-02:** add a CI matrix over `["3.11", "3.14"]` with
   `fail-fast: false`, leaving `requires-python` and the ruff/mypy pins on the 3.11 floor.
   `.github/**` is protected config (`pipeline.json:17`), so a person applies it; the ready
   file is in the session scratchpad as `sdlc-check.proposed.yml` (YAML-validated, 4 steps,
   matrix `["3.11", "3.14"]`). Side effect to accept: `check-feature.sh --run` then executes
   once per interpreter, so the feature-report binding check runs twice per PR. This gap stays
   open until the workflow file carries the matrix.
2. **Zero commits — open.** `git log` exits 128 ("branch 'main' does not have any commits
   yet"); `git ls-files` returns 0 files; the whole tree is untracked. The GitHub remote
   `Yaima/positronic-mind` is likewise empty (`gh repo view --json isEmpty` → `true`,
   `defaultBranchRef.name` → `""`, `git ls-remote --heads origin` → no output), so there are
   no CI runs yet (`gh run list` → empty). Consequences: git rules that diff against
   `base_branch: "main"` (`pipeline.json:48`) have no baseline; the `sdlc-check` gate has never
   executed; `/estimate` has no history to calibrate on. Commit and push are human actions
   (`commit_mode: human`, `pipeline.json:58`).

**Fixed, recorded for the next rerun**

3. `security_cmd` now passes. `python -m pip_audit` exits **0** ("No known vulnerabilities
   found"). On 2026-10-29 it exited 1 on four vulns in the venv's own pip 26.1.1
   (PYSEC-2026-196, PYSEC-2026-3721). Fixed in both places it runs: the venv was upgraded to
   pip 26.3, and CI now does `python -m pip install -U pip` before the editable install
   (`.github/workflows/sdlc-check.yml:17`). It still skips `positronic-mind` itself as not
   found on PyPI, which is expected for an unpublished package and not a failure.

**Project instructions vs. disk**

No disagreements. Every Repo-facts line checks out against disk: stack and src-layout
(`pyproject.toml:10,18-19`); pytest / ruff / mypy strict (`pyproject.toml:16,21,27`); no DB,
queues, or API layer (`pyproject.toml:13` — `dependencies = []`); one subpackage per layer (all
seven present under `src/positronic_mind/`); `tests/` flat with one file
(`tests/test_package.py`); `eval/suite/` and `eval/experiments/` holding only `.gitkeep`;
`values.example.md` present and `values.md` gitignored (`.gitignore:2`); proposal,
research-review, and decisions all present under `docs/`; no entry point; base branch main with
PRs via gh (`pipeline.json:48,57`).

**Not found / unmeasurable**

- `docs/pipeline/pipeline.json` still matches `pyproject.toml` on every command — no config
  change to propose. `coverage_cmd` and `min_coverage_percent: 0` are deliberately unset.
- No feature exists in `docs/features/` (`.gitkeep` only), so there is no `state.json` to read
  and no calibration data for `/estimate`.
- `docs/pipeline/agent-runs.log` holds a single `event=stop agent=unknown` line from
  2026-10-30; `agent=unknown` means the hook could not resolve a role name for that run.
- No `conftest.py`, no fixtures, no CHANGELOG, no Makefile, no Dockerfile, no `board/`.
