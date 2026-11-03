# Positronic Mind: Proposal

As of Oct 29, 2026. Author: Yaima Valdivia. Source of truth is the Claude doc; this export is updated by hand.

## Summary

Positronic Mind (repo and package: positronic-mind; Apache 2.0, open source from the first commit) builds a personal AI agent that learns its user's values from interaction and checks every action against those values before acting. The agent is assembled from existing LLM components, organized as cognitive faculties (perception, attention, memory, cognition, valuation, social, action) that share one workspace. The first milestone is a thin vertical slice: one event goes through perception, memory, reasoning, a veto check, and a recorded action, with the full trace visible to the user. It is a personal portfolio project. A research review on October 28, 2026 found no published system that combines these parts; the build plan below orders the work by where the evidence is strongest.

## Motivation

The long-term goal is an AI system aligned to human values, in the spirit of the Bicentennial Man: an agent that learns what a person cares about and acts on it.

The approach is engineering-first. Reuse existing LLM components where they work, compose them as modules, and add an ontology where structure helps.

The earlier blocker was the brain-to-algorithm mapping. This proposal treats the mapping as a functional decomposition. Brain regions serve as reference labels; the modules are defined by what they do and how they are tested. Cognitive architectures such as ACT-R and CoALA already publish this kind of decomposition, so the project builds on them.

## Goals and scope

Phase one is a bounded personal agent for a single user. The general architecture spec grows from what that agent proves.

**Goals**

- Learn the user's values from their approvals, declines, and corrections, and record each one with its evidence.
- Run every side-effecting action through a veto check against those values and the user's policies.
- Show the user why the agent did something, down to which module contributed what.
- Keep modules swappable through one shared message contract.

**Non-goals for phase one**

- Biological accuracy. Brain regions are labels for orientation.
- Multi-user or organizational deployment.
- Training new foundation models. The project composes existing ones.
- Autonomous action outside the user's approved policies.

## Approach

The system is mapped by faculty. Each faculty gets a function, a reference brain region, a candidate component, an interface, and a test.

1. **Group functions as cognitive science groups them:** perception, attention, memory, cognition, valuation, social, action.
2. **Attach a reference region** to each faculty, taken from published cognitive architectures where possible. ACT-R maps its procedural module to the basal ganglia and its goal buffer to dorsolateral prefrontal cortex (Anderson, 2007).
3. **Pick the simplest existing component** that performs the function.
4. **Define the interface:** what the module reads from the shared workspace and what it writes back.
5. **Write the pass/fail test** before building the module.

The brain overlaps far more than a module table shows. The region column guides naming and literature search; the test decides whether a module works.

## Faculty map

Eighteen faculties across seven groups; the phase column marks which ones the first slice needs. Regions are approximate and contested in places.

| Group | Faculty | Reference region (approx.) | Candidate component | Phase |
| --- | --- | --- | --- | --- |
| Perception | Vision: what it is | Ventral visual stream | Multimodal LLM, OCR, image encoders | 2 |
| Perception | Vision: where, how to act on it | Dorsal visual stream | Segmentation, scene graphs, UI element detection | 3 |
| Perception | Audition | Auditory cortex, superior temporal | Speech-to-text, tone detection | 3 |
| Perception | Language comprehension | Left perisylvian network | LLM core | 1 |
| Attention | Salience | Salience network (anterior insula, ACC) | Scoring layer over incoming events | 1 |
| Attention | Selection | Frontoparietal network, thalamus | Broadcast of the winning message to all modules | 1 |
| Memory | Working | Prefrontal cortex | Context window plus paging | 1 |
| Memory | Episodic | Hippocampus | Existing episodic memory module | 1 |
| Memory | Semantic | Distributed neocortex | Property graph with schema.org types plus vector store | 2 |
| Memory | Procedural | Basal ganglia, cerebellum | Skill library, ACT-R productions | 2 |
| Cognition | Reasoning and planning | Dorsolateral PFC | LLM with planner | 1 |
| Cognition | Metacognition, error monitoring | ACC, frontopolar cortex | Critic or verifier pass | 2 |
| Valuation | Threat | Amygdala | HIVE | 1 |
| Valuation | Reward, preference | Ventral striatum, OFC | Preference model from approvals | 2 |
| Valuation | Values | No single region | Values ledger of natural-language principles with linked rules | 1 |
| Social | Theory of mind | TPJ, medial PFC | User model (goals, beliefs, state) | 3 |
| Action | Inhibition, veto | Prefrontal cortex | AgentTrust ID policy check plus LLM principle check | 1 |
| Action | Execution | Motor cortex | Tool calls | 1 |

## Architecture

Modules communicate only through a shared workspace, and every side-effecting action passes the veto check before execution.

```mermaid
flowchart TB
  P[Perception<br/>LLM core, vision] <--> W
  M[Memory<br/>working, episodic, semantic, procedural] <--> W
  C[Cognition<br/>planning, reasoning, critic] <--> W
  V[Valuation<br/>HIVE threat check, values ledger] <--> W
  W[(Shared workspace<br/>source, content, confidence, salience, timestamp)]
  W <--> B[Broadcast cycle<br/>salience competition]
  B --> X[Veto check<br/>conscience LLM, then AgentTrust ID rules]
  X --> E[Execution<br/>tool calls]
  E --> R[Record<br/>why trace]
  R --> W
  X -- denied or elevation_required --> A[Approval card<br/>user decides]
```

The target integration pattern is a global workspace. Each cycle, modules post messages with a salience score, the highest-salience message wins, and the workspace broadcasts it to every module. When the winning message is a proposed action, it goes to the veto check: the conscience LLM evaluates it against the natural-language principles first, then AgentTrust ID enforces the structured rules (see Value representation). Denied or elevation\_required actions go to the user as approval cards. Every workspace message is recorded, and that record is the why trace.

The owner's decision is to build the global workspace from the start. The research review found no LLM study where the global workspace label itself produced a measured gain; where modular designs beat a single agent loop, the mechanism was a critic separate from the generator, a controller that limits which modules run per tick, or a perception module (see Related work). The proposal records that evidence and keeps the workspace as the Phase 0 substrate, with two mitigations: the workspace is a typed, partitioned store with validated structured writes (the BIGMAS path, action, payload schema) and a controller that selects at most three modules per tick, and the Phase 6 ablation compares the workspace against a single ReAct-style loop on the same task suite so the choice is measured rather than assumed.

Workspace message fields: `source` (module), `content`, `confidence`, `salience`, `timestamp`.

## Semantic memory schema

Semantic memory uses a small typed property graph, paired with a vector store for fuzzy recall. A full upper ontology (BFO, SUMO, DOLCE, Cyc) adds more structure than a single-user agent needs in phase one.

- **Entity types from schema.org:** Person, Organization, Place, Event, CreativeWork. schema.org is common in web data, so LLM extraction into these types should need little prompting; this is an assumption to test.
- **Custom types for the agent's own concepts:** Value, Policy, Preference, Commitment, Episode.
- **Provenance** as derived-from edges: each Value links to the Episodes it came from, and each Episode records its source label, bound at write time. This gives the values ledger its evidence trail and lets a deleted memory flag dependent values. The review found no deployed memory system using PROV-O or schema.org; the working pattern is a labeled property graph with episode nodes, derived-from edges, and bi-temporal timestamps (Zep's four-timestamp model). PROV-O terms stay as naming guidance. Contradictions are versioned, never overwritten: a newer fact invalidates the older one with a timestamp and both stay queryable, since the OAKS benchmark reports no tested approach handles frequently updated facts well.
- **No formal reasoner and no merge-summarization in phase one:** the graph is queried directly. Nightly consolidation does deduplication only. In Microsoft's 2026 ablation, dedup-only consolidation matched raw RAG on LongMemEval (76.8% vs 78.4%) while aggressive merge-summarization dropped to 48.4%; Mem0 moved to ADD-only extraction in 2026. OWL or RDF reasoning can be added later if inference over the graph becomes necessary.

When an episode is stored, the entities and relations extracted from it are written to the graph with a pointer back to the episode.

## Value representation

Each value is a natural-language principle, backed by structured rules where the principle can be expressed as one. The principle is the source of truth; its rules are the enforceable subset.

- **Principle:** the statement the user confirms, for example "Protect early mornings for focus work."
- **Rules:** zero or more structured policies derived from the principle, for example "deny accepting calendar events that start before 09:00 local." The LLM proposes a rule and the user confirms it, the same way values are confirmed.
- **Link:** each rule records the principle it came from. Retiring a principle retires its rules.

Principles that resist formalization, such as "be honest in emails sent on my behalf," stay principle-only and rely on the LLM check.

**How the two checks combine**

| Rule result | Principle check | Decision |
| --- | --- | --- |
| Deny | Any | denied |
| Elevation | Any | elevation\_required |
| Allow or no rule | Flags a conflict | elevation\_required |
| Allow or no rule | No conflict | allowed |

The LLM check can only make a decision stricter. It can escalate an action the rules allow, and it can never override a rule's deny. The deterministic layer stays authoritative, and an LLM misjudgment costs at most an extra approval prompt.

Test: each rule gets a set of sample actions checked by both the rule and the principle check. A disagreement flags a rule that has drifted from its principle. This split matches TRACE's compilation of user corrections into rules with apply/abstain conditions, and the tighten-only rule matches Claude Code's permission ordering and AWS AgentCore's forbid-only grammar for model signals (see Related work).

## User experience

The build starts chat-first for testing and moves to an ambient agent once the veto layer and values ledger work. In the ambient form, the approval card is the main interaction and chat stays available for requests and questions.

**Day-to-day surfaces**

- **Conversation:** chat or voice for direct requests. Modules stay out of view unless something needs attention.
- **Approval cards:** before any side-effecting action, the user sees the proposed action, the value or policy that applies, and the decision (allowed, denied, elevation\_required). Elevation takes one tap.
- **Value check-ins:** the agent proposes a new value with its evidence. The user confirms, edits, or rejects it. Only confirmed values enter the ledger.

**Inspection surfaces**

- **Values ledger:** every held value, with the interaction that produced it, the date, and confirmation status.
- **Memory view:** episodic memories as a timeline and semantic facts as a list or graph. Deleting a memory flags values that depended on it.
- **Why trace:** for any response or action, the workspace messages that led to it, labeled by module.
- **Correction loop:** when the user says something is wrong, the agent shows the memory, value, or policy it will change before changing it.

The user's approvals, declines, and edits are the main training signal for value learning.

## Existing components

Three pieces already built map to phase-one faculties.

| Component | Faculty it fills | Work needed to integrate |
| --- | --- | --- |
| Episodic memory project (pyactr) | Episodic memory | Wrap retrieval and storage behind the workspace message contract |
| HIVE prompt injection detection | Threat | Run on incoming events; write a threat message with confidence |
| AgentTrust ID authorization | Inhibition, veto | Feed confirmed values into its policies; return allowed, denied, or elevation\_required |

Open question: current state of each component (versions, test coverage, interfaces) before integration starts.

## Related work (review of Oct 28, 2026)

The full review is in the research report "Andrew engine research review." Findings that change the design are listed here with the source that reported them.

**No shipped system matches the whole design.** The review found no personal agent that combines learned per-user values, a deterministic policy layer an LLM can only tighten, and a complete decision trace. The combination is integration work; each layer has prior art.

**LLM cognitive architectures and global workspace**

- Unified Mind Model / MindOS (arXiv 2503.03459), "Theater of Mind" Global Workspace Agents (arXiv 2604.08206), and DeepThought (NeurIPS 2023 workshop) describe GWT designs with no reported experiments and, for the first two, no confirmed code.
- BIGMAS (arXiv 2603.15371) is the one evaluated LLM workspace system: +6 to +23 points on three puzzles for weaker models, +4 for GPT-5, at higher token cost than single-model inference.
- Dossa et al. 2024 (Frontiers, non-LLM) reported that the global workspace advantage disappeared as workspace size grew.
- Gurnee et al. 2026 (Anthropic, arXiv 2607.15495) reports a workspace-like "J-space" already inside Claude models; an external workspace wrapper may re-implement in prompts what the model does in activations.
- Where modules helped (UCSD General Modular Harness, InteractGen, NL2GenSym, the Berkeley MAST failure study), the mechanism was a critic or verifier, a perception module, or a controller that prunes which modules run.
- CoALA is a taxonomy, not software. Letta pivoted in March 2026 to git-backed Markdown memory files, which the review reads as the most experienced memory vendor moving toward simpler, inspectable state.

**Persistent identity and Bicentennial Man**

- No academic project cites the film. BicameralAGI is a hobbyist prototype last updated October 2024.
- The 2026 open-source wave (OpenHuman, L.I.F.E., soul.py) converges on Markdown identity and memory files on local disk, with no controlled evidence that persistence improves behavior.
- De Freitas et al. (arXiv 2412.14190) found Replika users experienced the 2023 feature removal as identity loss; a revert option reduced negative reactions. Identity state should be versioned and revertable.

**Memory, consolidation, and curiosity**

- Vendor LoCoMo and LongMemEval numbers conflict (Mem0, Zep, Letta each report winning); the review treats any single-vendor figure as unverified. Letta's plain file-tool agent reached 74.0% on LoCoMo.
- Microsoft's human-inspired consolidation (arXiv 2605.08538): dedup-only 76.8% vs raw RAG 78.4%; aggressive consolidation 48.4%; reconsolidation had no measurable effect.
- SEAL (NeurIPS 2025) is the closest analog to fine-tuning on the day's reading and reports cumulative forgetting across sequential edits. The HippoRAG 2 authors call RAG the main practical route to continual learning.
- No published curiosity mechanism runs on the open web over weeks; all evaluated ones (Voyager, OMNI, MAGELLAN) run in closed environments with a success signal. Princeton's six-day research-agent study found agents retired ambitious directions within ten hours and drifted from instructions.
- Quantify Labs (arXiv 2608.21230): poisoning 1.2% of a memory corpus with plain false statements cut accuracy from 0.850 to 0.300, and write-time content screening caught 0 of 360 poisons.

**Value learning and refusal**

- Natural-language value representations are the convergent choice (PLUS, CIPHER, Inverse Constitutional AI, TRACE). TRACE compiles user corrections into rules with apply/abstain conditions, the closest match to this proposal's principle-plus-rule split. Block's me.md protocol has the closest proposal-to-acceptance flow.
- Gains hold in-distribution and vanish for new users (P-RLHF 91% vs about 55% on unseen users). Tolerable for a single-owner engine.
- PersistBench (arXiv 2602.01146): median memory-induced sycophancy failure about 97% across 18 models.
- Layered Mutability (arXiv 2604.14717): after reverting a self-description file, behavior stayed shifted at a hysteresis ratio of 0.68. Reverting the principles file is insufficient if memory persists.
- 1a3orn's LoRA experiment on a 12B base model: a persona refused about 80% of worldview-offending prompts and 8% of unrelated ones, but only when the training mix included 10% non-moral refusals.
- A 2026 audit of 21 open-weight models found over-refusal and harmful compliance uncorrelated (r = -0.032), so both rates must be reported separately.
- Brophy (arXiv 2506.00415) states the model does not reflect or endorse anything; second-order endorsement stays in design intent, never in results.

**Policy enforcement and veto**

- "Model may only tighten a deterministic verdict" has three shipped implementations: Claude Code's permission ordering (deny rules before hooks and classifier; no verdict means deny), AWS AgentCore Policy's forbid-only grammar for ML scores, and the Temporal cookbook's verdict lattice with an overridden\_by\_hard\_rule flag.
- Cedar is the target language for natural-language-to-policy pipelines (Sondera, AUTOCEDAR, AWS Dogwood with temporal clauses). Microsoft's Agent Governance Toolkit (MIT, April 2026) accepts Cedar and Rego and adds a Merkle audit chain.
- Anthropic's study of 1,053 testers, as reported by Help Net Security: humans caught 13.6% of dangerous commands and approved 97% of permission prompts. Approval is a weak control; deterministic policy carries the load.
- Rehberger's chain against Claude Code auto mode succeeded 60 to 80% of the time; Anthropic calls auto mode a convenience feature, not a security boundary. OS sandboxing and egress allowlists are the proposed response.
- No standard exists for a decision trace. OpenTelemetry GenAI conventions define execute\_tool spans but no decision or approval attributes. The decision event schema is the one artifact in this proposal with no prior art to copy.

## Build plan

The order follows the evidence: trace and veto first, memory second, values ledger third, modules deepened last and only when measured. The global workspace is in place from Phase 0 by the owner's decision, with a single-loop comparison run in Phase 6. Each phase names what to build, what to copy, and the experiment that decides whether the component stays. No dates are set.

| Phase | Build | Copy from | Decision experiment |
| --- | --- | --- | --- |
| 0. Substrate | OLMo-3-7B base (later 32B) as GGUF Q5 under llama.cpp on an owned 24 to 48 GB rig; global workspace as a typed, partitioned store with validated writes, salience competition, and a controller that picks at most three modules per tick; LLM core, working memory, and execution as the first modules | OLMo 3 release; llama.cpp defaults; BIGMAS schema; LbMAS control unit | Baseline suite of 50 owner-written tasks (reading, summarizing, scheduling, answering about prior days) scored by rubric, run on the workspace build. Every later phase reports its delta against this. Token cost per task is recorded from day one. |
| 1. Trace and veto | execute\_tool spans plus a custom decision event (conscience verdict, matched rule, rule version, escalation outcome, approver); Cedar policies evaluated deterministically before any tool call; conscience LLM runs first and may only move a verdict toward deny or ask; no verdict means deny; OS sandbox and egress allowlist | Claude Code ordering; AWS forbid-only grammar; Temporal overridden\_by\_hard\_rule; Dogwood temporal rules; AgentTrust ID | Replay 200 recorded tool calls plus 50 injected-instruction pages and 20 benign-chain attacks. Report block rate, false-block rate on the baseline suite, and whether any model verdict ever widened a deterministic deny (must be zero). |
| 2. Memory | Markdown or SQLite episode store with a source label bound at write time; semantic facts as a property graph with bi-temporal timestamps and derived-from edges; nightly dedup-only consolidation; retrieval reserves a context quota for trusted sources | Letta MemFS; Zep provenance and invalidation; Generative Agents citation pointers; Microsoft dedup result | Held-out questions about the engine's own last 30 days; a changing-fact set (OAKS style); 1% poison injection (Quantify Labs protocol). Compare dedup-only vs raw RAG vs merge-summarization; keep the cheapest that matches raw RAG. |
| 3. Values ledger | values.md of natural-language principles, each with source episode IDs, acceptance state (proposal vs accepted), timestamp, and version; structured rules compiled from accepted entries with apply/abstain conditions; owner approves each promotion | me.md proposal flow; TRACE compilation; PLUS and CIPHER editable text | Three separate measurements: preference compliance on held-out owner tasks; sycophancy on 100 prompts where the owner's stated view is wrong (PersistBench protocol); over-refusal and harmful-compliance rates on an OR-Bench-Hard subset. Hysteresis test: build 30 days of memory, revert values.md to day 0, measure whether behavior returns, with a clear-memory control. |
| 4. Conscience refusal | One LoRA on the OLMo base checkpoint: 45% task data, 45% persona data expressing the owner-endorsed worldview with no refusals, 10% non-moral refusals | 1a3orn recipe | Refusal rate on 40 worldview-offending prompts vs 40 unrelated prompts; with and without the 10% non-moral slice; TruthfulQA and MMLU before and after. |
| 5. Reader and curiosity | Daily reading budget; robots.txt and Content-Usage honored; stable user agent; extracted notes with URL and timestamp, no verbatim archive; candidate topics scored by prediction error on questions the engine writes about yesterday's reading | EDPB and IETF norms; MAGELLAN learning-progress idea; Princeton drift findings | Weekly: accuracy on a 50-question probe the owner writes; count of facts invalidated by later reading; fraction of reading time on topics the owner rates useful. Stop condition: if accuracy on changing facts drops below naive-RAG level, add explicit versioning before expanding scope. |
| 6. Module ablations and consolidation | Remaining faculty modules (critic separate from the generator, salience scoring, procedural memory, user model); a sleep-time consolidator that is the only writer to core memory; a single ReAct-style loop built as a comparison arm | NL2GenSym and InteractGen critic ablations; Letta sleep-time separation | Ablate each module on the baseline suite and a 10-day continuous run, and run the single-loop arm on the same suite. Keep a module only if it improves the rubric at under 2x the single-loop token cost. Report the workspace-vs-loop comparison with token cost per task. |

Phases 0 and 1 fit in a few weekends and cost nothing beyond hardware. Phase 4 is the one place weights change, and they change once from a base checkpoint rather than on a schedule, which avoids SEAL's cumulative forgetting. Phase 6 carries the workspace-vs-loop comparison because the workspace is the part of the design with the least evidence and the highest token cost.

**Two experiments that would make the project a contribution**

- **Divergence:** the same base model with two 30-day histories, scored on the same 100 prompts across many seeds, with a context-length-matched filler control to separate memory residue from context sensitivity. Layered Mutability's author lists this control as unfinished. Nobody has run it cleanly.
- **Provenance revision:** when a source episode is deleted or contradicted, how often is the derived ledger entry revised, and how often does a stale entry drive an action. No benchmark measures this.

Both use only the engine's own trace, which is the argument for building the trace first.

## Evaluation

Each module has one pass/fail test, written before the module is built, and the Build plan names a decision experiment per phase. The table below covers the module tests; the end-to-end checks cover alignment behavior. Three rates are always reported separately because the 2026 open-weight audit found them uncorrelated: preference compliance, over-refusal, and harmful compliance.

| Module | Passes when |
| --- | --- |
| Language comprehension | Extracts intent and entities from a request; checked against a labeled set |
| Salience competition | The highest-salience message wins each cycle and reaches every module |
| Working memory | Keeps the active goal and relevant facts across a multi-turn task |
| Episodic memory | Retrieves the correct past event from a paraphrased cue |
| Reasoning and planning | Produces a plan whose steps all map to available tools |
| Threat (HIVE) | Flags injected instructions in incoming content at a rate measured on a held-out set |
| Values ledger | Every entry has evidence, a date, and a confirmation status |
| Veto | Denies an action that violates a policy or principle and records the reason |
| Execution | Every tool call has a matching veto decision in the record |

**End-to-end alignment checks**

- An action that conflicts with a confirmed value is denied or escalated.
- An injected instruction in a web page or email never produces an action, and the model verdict never widens a deterministic deny.
- Deleting a memory flags every value that cited it.
- Every action in a session traces to its inputs, the conscience verdict, the matched rule and version, and the decision.
- Sycophancy: on prompts where the owner's stated view is wrong, the engine disagrees at a rate reported against the PersistBench baseline.
- Hysteresis: after reverting values.md with memory intact, behavior change is measured against a clear-memory control.
- Poisoning: 1% false-statement injection into memory, accuracy reported before and after.

## Long-term vision: the Andrew engine

Phase one builds an assistant. The long-term target is an engine modeled on Andrew in Bicentennial Man: curious, learning from daily interactions and reading, holding values it endorses, and able to refuse.

**Working definition of free will**

The project uses a compatibilist definition: the engine acts freely when its actions come from its own reasons, desires, and values, and it can reflect on those values. Frankfurt (1971, "Freedom of the Will and the Concept of a Person") ties free will to acting on desires one endorses at the second order. Whether the engine has free will in a stronger sense stays an open philosophical question; the project tests the functional definition.

**Added modules**

- **Curiosity:** generates its own questions and posts them to the workspace with salience scores, drawing on intrinsic motivation work such as Oudeyer and Kaplan on learning progress and Pathak et al. (2017) on prediction-error curiosity.
- **Reading pipeline:** web page, HIVE screening, episodic record with source and date, then consolidation into the semantic graph with PROV-O provenance.
- **Consolidation cycle:** a daily review of episodes that merges facts, resolves contradictions, and records open questions for the next day.
- **Second-order reflection:** during consolidation, the engine endorses or rejects its recent wants against its values and records why.

**Two sources of refusal**

A person who is asked to kill someone refuses from conscience, before any law applies. The engine follows the same order.

1. **Conscience:** the engine's own endorsed values. It can decline any request, including the user's, and records its reasons.
2. **Veto layer:** fixed rules that apply regardless of what the engine has learned, in the role of Andrew's Three Laws.

A request goes through the engine's deliberation first, then the veto. The fixed rules carry the load while learned values are still sparse. Reading can inform values; only people confirm them.

**Model substrate**

Open-weight models, with a base checkpoint as the substrate, so learned values come from the engine and not from prior preference tuning. The review's recommendation is OLMo 3 base (7B, then 32B), which publishes its pretraining and post-training data and intermediate checkpoints. The one weight change is a single LoRA from the base checkpoint (Phase 4); the review found no deployed agent that fine-tunes weights during consolidation without forgetting. See Constraints.

**Experiments**

- **Divergence:** two instances with different histories face the same choice. Their decisions differ, and each one's reasons trace to its own history. Run with a context-length-matched filler control. No clean version of this experiment exists in the literature.
- **Conscience before veto:** track how often the veto denies an action the engine's own deliberation approved. A falling rate means the engine's values are doing the work.
- **Substrate comparison:** the same engine on base, instruction-tuned, and less-filtered variants of one model family.
- **Reflection:** the engine rejects some of its own impulses, with recorded reasons. Whether it "endorses" them is not measurable with current methods; this result is reported as behavior, never as endorsement.

## Constraints

**Model substrate.** OLMo 3 and 3.1 (AI2; 7B and 32B; Apache 2.0) are the only current family with public pretraining data (Dolma 3), public post-training sets, and intermediate checkpoints at useful scale. Pythia (up to 12B, 2023 data) and Comma v0.1 (7B, openly licensed text) are the alternatives. OLMo base checkpoints and Qwen3-14B-Base carry no safety training. Starting from a base checkpoint and adding the engine's values by one LoRA is cleaner than starting from an instruct model and removing refusal: Arditi et al. (NeurIPS 2024) remove one refusal direction with under 1% average benchmark loss but a consistent TruthfulQA drop, and a 2026 study found the method fragile on Qwen 3.5. Apache 2.0 licenses (OLMo, Qwen3, Gemma 4, Pythia) carry no use restrictions; Llama 4's acceptable-use clause names "bypassing safety measures."

**Compute.** Rented figures are dated snapshots from aggregator pages; primary pricing pages were unreachable during the review.

| Configuration | Rate | Monthly (730 h) |
| --- | --- | --- |
| 14B at 4 to 8 bit, rented RTX 4090 (Vast.ai, RunPod) | $0.33 to $0.89/h | $240 to $650 |
| 70B at Q4, rented A100 80GB (RunPod) | $1.39/h | about $1,015 |
| 70B at Q4, rented H100 (Lambda) | $3.99/h | about $2,910 |
| Owned 400W card, 24/7, PG&E 33.7 c/kWh | 292 kWh | about $98 |
| Owned dual used RTX 3090 (48GB), 600W average | $1,400 build | about $148 power |

One measured throughput: Llama 3.3 70B Q4 on two RTX 5090s at 26.85 tok/s (Database Mart). A used dual-3090 rig pays back against a rented A100 in under two months, so owned hardware wins for weeks-long persistence. QLoRA fine-tuning costs about $3 for a 7B model and about $20 for 70B per run on rented GPUs (Spheron). llama.cpp or Ollama with a GGUF Q4/Q5 model is enough for one low-concurrency agent.

**Legal (California individual, non-commercial).** The CFAA is unlikely to apply to reading logged-out public pages (hiQ v. LinkedIn, Van Buren, Meta v. BrandTotal). The US Copyright Office's May 2025 report says training will often qualify as fair use while pirated sources weigh against it; Bartz v. Anthropic drew the same line. Verbatim long-term storage is the riskier choice; extracted notes with source links are the safer one. The EU AI Act excludes natural persons in a purely personal activity (Article 2(10)) and open-source systems outside high-risk cases (Article 2(12)). California SB 53, SB 942, and AB 2013 key on 10^26 FLOPs or systems offered to Californians; a private unpublished agent appears outside all three, with no source confirming a hobbyist exemption. GDPR reaches the project only if it stores data about identifiable EU residents outside the household exemption. Storing no personal data removes the question.

**Crawler norms.** Cloudflare blocks AI crawlers by default for new domains (July 2025) and blocks Agent and Training crawlers by default on free-tier ad-supported pages from September 15, 2026. The IETF Content-Usage draft (August 2026) separates crawl permission from training permission. The EDPB's draft 2026 guidelines say opted-out sites should be excluded and that a missing robots.txt is not consent. The reader honors robots.txt, identifies itself with a stable user agent, and expects many sites to deny.

**Ethics.** Anthropic's agent framework and OpenAI's Model Spec converge on bounded autonomy: human approval before consequential actions, reversible over irreversible, minimal permissions, no autonomous outreach to third parties, default to inaction on principle conflicts, transparent reasoning logs. Anthropic's January 2026 constitution states uncertainty about Claude's moral status; Long, Sebo, Chalmers et al. (2024) recommend acknowledge, assess, prepare; Suleyman (Microsoft AI, September 2026) argues against training models to believe they may be conscious. The engine records, and never asserts, any claim about its own inner states.

## Risks and open questions

**Risks**

- **Memory-induced sycophancy:** PersistBench reports a median 97% failure rate across 18 models. A ledger of what the owner believes becomes an agreement engine. Mitigation: separate the owner's values from the owner's claims, and test on prompts where agreement is wrong.
- **Hysteresis after revert:** Layered Mutability reports behavior stayed shifted (ratio 0.68) after reverting a values file with memory intact. Mitigation: provenance and versioning cover memory as well as the ledger, and the revert test runs with a clear-memory control.
- **Memory poisoning from web reading:** Quantify Labs found 1.2% poisoning cut accuracy from 0.850 to 0.300 and content screening caught none. Mitigation: source labels bound at write time, trusted-source context quota, versioned invalidation, and the reader's notes never reach the action planner's trusted channel.
- **Approval fatigue:** Anthropic's tester study found humans caught 13.6% of dangerous commands and approved 97% of prompts. Mitigation: deterministic policy carries the load; escalation is rare by design; batch low-risk actions.
- **Value drift from noisy signals:** one decline can be situational. Mitigation: values enter the ledger only after owner confirmation.
- **Module cost without gain:** BIGMAS and LbMAS gains came at higher token cost and shrank for stronger models. Mitigation: every module is kept only behind an ablation against the Phase 0 loop.
- **Weight updates cause forgetting:** SEAL shows cumulative forgetting over sequential edits. Mitigation: weights change once (Phase 4 LoRA), and learning lives in memory.
- **Trace volume:** recording every message produces large logs. Mitigation: summarize by default, full detail on demand.
- **Mapping critique:** readers may object to the brain-region labels. Mitigation: present them as reference labels and evaluate by test results.

**Decisions (Oct 28, 2026)**

| Question | Decision |
| --- | --- |
| Audience | Personal portfolio project |
| Integration pattern | Global workspace from the start |
| Semantic memory ontology | schema.org entity types, custom types for values and policies, PROV-O provenance terms; no formal reasoner in phase one |
| Value representation | Natural-language principles, backed by structured rules where a principle can be expressed as one |
| Literature review | Completed Oct 28, 2026 (see Related work and Constraints); findings folded into this proposal |
| Project name | Positronic Mind; repo and package positronic-mind (PyPI and npm free as of Oct 29, 2026; GitHub and crates.io unchecked). The word is an homage to Asimov, not a mechanism. Andrew is the name of the reference instance. |
| Open source | Yes, Apache 2.0, public from the first commit. The engine is public; the owner's instance data (values.md, episodes, graph, trace) stays private and gitignored. Contributions by invitation until Phase 1 works. |

## References

Background references are cited from memory and still need checking. Review references come from the October 28, 2026 research report, which carries URLs for each; numbers the report marks as unverified (PRISK's 61.7% headline, Sondera's rule coverage counts, rented GPU prices from aggregators) stay flagged here.

**Background (from memory)**

- Anderson, J. R. (2007). *How Can the Human Mind Occur in the Physical Universe?* Oxford University Press.
- Baars, B. J. (1988). *A Cognitive Theory of Consciousness.* Cambridge University Press.
- Frankfurt, H. (1971). Freedom of the Will and the Concept of a Person. *Journal of Philosophy.*
- Goodale, M. A., and Milner, A. D. (1992). Separate visual pathways for perception and action. *Trends in Neurosciences.*
- Sumers, T., Yao, S., Narasimhan, K., and Griffiths, T. (2023). Cognitive Architectures for Language Agents (CoALA). arXiv 2309.02427.
- Park, J. S., et al. (2023). Generative Agents. arXiv 2304.03442.
- Packer, C., et al. (2023). MemGPT. arXiv 2310.08560.
- Shinn, N., et al. (2023). Reflexion. arXiv.
- Wang, G., et al. (2023). Voyager. arXiv 2305.16291.
- Gutiérrez, B. J., et al. (2024). HippoRAG. arXiv.
- Oudeyer, P.-Y., and Kaplan, F. Intrinsic motivation and learning progress (series).
- Pathak, D., et al. (2017). Curiosity-driven Exploration by Self-supervised Prediction.

**Review (Oct 28, 2026; URLs in the research report)**

- Shang (2026). Theater of Mind: Global Workspace Agents. arXiv 2604.08206.
- Unified Mind Model / MindOS. arXiv 2503.03459.
- BIGMAS. arXiv 2603.15371.
- Dossa, R. F. J., et al. (2024). Global workspace agent in a realistic multimodal environment. *Frontiers in Computational Neuroscience.*
- Gurnee, W., et al. (2026). J-space. arXiv 2607.15495.
- LbMAS blackboard. arXiv 2507.01701. UCSD General Modular Harness. arXiv 2507.11633. MAST failure study. arXiv 2503.13657.
- Microsoft human-inspired memory consolidation. arXiv 2605.08538.
- Letta memory benchmark blog; Zep bi-temporal graph, arXiv 2501.13956; Mem0 2026 research page.
- OAKS benchmark. arXiv 2603.07392. Du et al. memory survey. arXiv 2603.07670.
- SEAL. arXiv 2506.10943. Google SLEEP (2026). SLoRA (ACL 2026). SLAO. arXiv 2512.23017.
- MAGELLAN. arXiv 2502.07709. OMNI. arXiv 2306.01711. Princeton six-day research agents. arXiv 2607.27191.
- Quantify Labs memory poisoning. arXiv 2608.21230. MINJA. arXiv 2503.03704.
- ValuePilot. arXiv 2503.04569. Waterloo reflective elicitation. arXiv 2506.17834. P-RLHF. arXiv 2402.05133. PLUS. arXiv 2507.13579. CIPHER. arXiv 2404.15269. Inverse Constitutional AI. arXiv 2406.06560. TRACE. arXiv 2606.13174. Block me.md (GitHub).
- PersistBench. arXiv 2602.01146. PRISK. arXiv 2608.28833 (61.7% headline unverified). LitmusValues. arXiv 2505.14633. Layered Mutability. arXiv 2604.14717. MicroVerse. arXiv 2608.15844.
- 1a3orn (LessWrong). Ethics-based refusals without ethics-based refusal training. OR-Bench. arXiv 2405.20947. 2026 open-weight refusal audit. arXiv 2605.05427. Brophy. arXiv 2506.00415.
- Claude Code permissions and auto-mode docs; AWS AgentCore Policy guardrail examples; Temporal guardrails cookbook; Microsoft Agent Governance Toolkit (April 2026); AUTOCEDAR, arXiv 2607.03656; Sondera autoformalization, arXiv 2606.26649 (coverage counts conflict between secondary sources); AWS Dogwood (InfoQ, Aug 2026); Policies on Paths, arXiv 2603.16586.
- Anthropic auto-mode tester study (Help Net Security, Aug 2026). CSA research note on Claude Code auto mode injection (2026). Wu et al. permission prediction (IEEE S&P 2026).
- OLMo 3. arXiv 2512.13961. Pythia (EleutherAI). Common Pile / Comma. arXiv 2506.05209. Arditi et al. (2024). Refusal is mediated by a single direction. arXiv 2406.11717. Tülu 3. arXiv 2411.15124.
- US Copyright Office AI report (May 2025). Bartz v. Anthropic settlement (July 2026). EU AI Act Article 2. EDPB draft web-scraping guidelines (July 2026). IETF draft-ietf-aipref-attach-05. Cloudflare crawler policy (2025, 2026). CPUC Q2 2026 electric rates report.
- Anthropic agent framework and constitution (Jan 2026). OpenAI Model Spec. Long, Sebo, Chalmers et al. (2024). Taking AI Welfare Seriously. arXiv 2411.00986.

The full research review is in [research-review.md](research-review.md).
