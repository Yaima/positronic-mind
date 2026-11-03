# Values ledger (example instance: Andrew)

This file is a fictional sample. A real instance keeps its ledger in `values/values.md`, which is gitignored.

Each entry is a natural-language principle the owner confirmed. `evidence` lists the episode IDs it was derived from. `rules` lists the structured rules compiled from it, each confirmed separately. Retiring a principle retires its rules.

---

id: V-0001
state: accepted
confirmed: 2026-11-01
version: 1
principle: Protect early mornings for focus work.
evidence: [ep-2026-10-30-014, ep-2026-10-31-002]
rules:
  - id: R-0001
    state: accepted
    cedar: forbid(principal, action == Action::"calendar.accept", resource) when { resource.start_local_hour < 9 };

---

id: V-0002
state: proposed
proposed: 2026-11-03
version: 1
principle: Be honest in messages sent on my behalf, even when a softer wording would be easier.
evidence: [ep-2026-11-02-009]
rules: []
note: Principle-only. No rule expresses this; the conscience check covers it.
