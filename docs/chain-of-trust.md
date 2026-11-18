# Chain of Trust (`C0–C4`), Verification Ladder, and Rigor Grades

> This document specifies the verification architecture and trust-chain model for [Meta DFM](https://metadfmp.drayker.org). It extends the metacognitive solver/prover design into verifiable engineering and knowledge provenance. It is an architectural specification and research model, not a deployed autonomous service.

---

## 1. Foundations: Why Trust Must Be Computed Per Use

In Dknowledge and Meta DFM, trust cannot be an immutable global label stamped onto an author, model, or module. A component proven sound under one boundary can fail silently when composed into a new caller or a new physical context. Five foundational principles govern the chain of trust:

1. **Trust grades move with evidence.** Confidence rises only as independent verification accumulates and decays or resets to zero whenever code, tests, commands, or declared assumptions change.
2. **Formal verification is strongest, yet still needs non-formal backing.** Mathematical proof (`D5–D7`) reaches the highest structural confidence (`C4`), yet formal specifications rest on assumptions about hardware, environments, and human intent outside the proof kernel. Independent simulated proof must therefore examine what was not formalized.
3. **Trust branches per use.** A module verified for its own internal function earns its own trust grade (`own(P)`). Reusing that dependency `D` in a downstream component `P` for a new purpose opens a separate trust branch (`use(P, D)`) that starts unverified (`C0`) until integration or consumer-contract checks explicitly test that specific link.
4. **Complete records, simple calculation.** Record every artifact, seed, dependency link, parallel fallback, refutation, and failure completely now so deterministic rules stay auditable today and richer graph analyses can operate over the same ledger later.
5. **Minimum simulation bar.** Where complete formal proof is infeasible, independent adversarial review establishes explicit unproven boundaries, failure modes, and a minimum confidence threshold calibrated to the task's rigor grade.

---

## 2. The Verification Ladder (`S`, `D0–D7`, and Simulated Proof)

Meta DFM separates architectural structure checks, behavioral proof rungs, and non-formal analytical backing:

| Rung | Kind | What it establishes |
|---|---|---|
| `S` | Structural gate | Schema, dependency map, module boundary, and invariant-shape conformance. Necessary hygiene, **never** counted as behavioral proof. |
| `D0` | Example tests | Concrete unit/integration tests that include at least one regression case failing prior to the change. Baseline floor for executable trust. |
| `D1` | Property tests | Generative invariants checked across randomized inputs with recorded seeds. |
| `D2` | Mutation testing | Faults injected into AST/bytecode must be detected and killed by the test suite, proving test strength rather than code coverage alone. |
| `D3` | Consumer contracts | Interface schemas, invariants, and compatibility assertions driven from downstream consumers' assumptions. |
| `D4` | Differential testing | Identical inputs evaluated across an older module version being replaced and the new replacement (or across independent parallel solutions). |
| `D5` | Model checking | Bounded or exhaustive finite-state transition exploration across concurrency, protocol states, or distributed edge cases. |
| `D6` | SMT & symbolic execution | Constraint solving and path exploration verifying properties across symbolic input classes. |
| `D7` | Formal verification | Machine-checked proof in a formal language or verifier, gated by a deterministic **anti-shortcut scanner** that rejects incomplete proofs or escape hatches (`sorry`, `sorryAx`, `admit`, `native_decide`, `assume false`, `axiom`, `{:axiom}`, `{:verify false}`, `ensures true`, `external_body`, `verifier::external`). |
| `simulated proof` | Analytical backing | Independent skeptical review identifying unstated assumptions, unproved boundaries, edge cases, and failure hypotheses (`confidence: low \| medium \| high`). Never substitutes for executable rungs. |

---

## 3. Computed Trust Levels (`C0–C4`)

Trust levels are derived strictly from the latest non-stale passing evidence. Declaring `trust` or `confidence` as manual input inside an order, artifact metadata, or non-simulation verdict is rejected as an integrity error.

| Level | Name | Evidence required |
|---|---|---|
| `C0` | Declared | Specification, design note, or module exists without passing `D0`. Default initial state for every new artifact and every untested dependency link. |
| `C1` | Example-tested | Passing `D0`. |
| `C2` | Strong | `C1` plus at least one passing `D1` (property), `D3` (consumer contract), or `D4` (differential) check. |
| `C3` | Mutation-proven | `C2` plus passing `D2` (mutation testing kills injected faults). |
| `C4` | Formally verified | `C1` plus at least one passing `D5`, `D6`, or `D7` check using a verified local toolchain free of proof shortcuts, together with any additional rungs required by the assigned rigor grade. |

Strict order: `C0 < C1 < C2 < C3 < C4`.

---

## 4. Per-Use Trust and Weakest-Link Propagation

Let `P` be a part or module with declared dependency set `depends_on(P)`:

- **`own(P)`**: the trust level earned by passing checks of `P` that carry no `uses` dependency claim.
- **`use(P, D)`**: for each `D` in `depends_on(P)`, the trust level earned by passing integration or consumer-contract checks on `P` that explicitly declare `uses: [..., D, ...]`. If no passing check tests `P` using `D`, then **`use(P, D) = C0`** — even when `D` itself has reached `C4` in isolation.

Effective trust propagates by the **weakest link** in the chain:

```text
effective(P) = min( own(P), min over D in depends_on(P) of min( use(P, D), effective(D) ) )
```

For a composite piece (`_final`), overall effective trust is `min(effective(P))` across all parts, exposing an explicit `limited_by` trace naming whichever term capped the chain (`own`, `use of D`, or upstream `D`).

**Content-addressed invalidation:** every evidence record binds SHA-256 digests of the source files, test targets, command strings, and `uses` lists. Any modification to code, tests, commands, or order scope invalidates prior acceptance immediately and drops that version's unverified delta back to `C0`.

---

## 5. Rigor Grades (`G0–G3`), Fail-Fast Execution, and Mission Trace

Higher strategic impact commands greater compute budget, stricter proof floors, and parallel exploration:

| Grade | Minimum `effective(P)` per part | Simulated proof requirement | Additional gates |
|---|---|---|---|
| `G0` | `—` | `—` | Exploratory work; structural sanity. |
| `G1` | `C1` | Optional | Independent skeptic verdict + passing composition checks (`_final`). |
| `G2` | `C2` | Required (`confidence >= medium`) | Five-dimension review (`correctness`, `error_attack`, `author_fidelity`, `vocabulary`, `operability`), mandatory `serves` mission trace, >= 2 parallel solutions unless exempted, `_final` skeptic gate. |
| `G3` | `C3` | Required (`confidence = high`) | All `G2` gates, mandatory `D4` differential verification when `replaces` is declared, >= 3 parallel solutions unless backed by an explicit owner decision in `serves`, plus an advisory prompt whenever a `D5–D7` toolchain is installed locally and unused. |

- **Fail-fast scheduling:** within each verification pass, checks execute strictly from cheapest to most expensive (`S -> D0 -> D3 -> D1 -> D4 -> D2 -> D6 -> D5 -> D7`) and halt at the first failure to conserve compute (diagnostic runs may pass `--all` to observe all rungs without recording acceptance).
- **Mission trace (`serves`):** every non-trivial part at `G2` and `G3` must bind explicitly to an authorized specification requirement or recorded design decision (`serves`), preventing local optimization from drifting away from actual human/system intent.

---

## 6. Parallel Solutions, Contingency Plans B & C, and Grounded Refutation

When rigor requires depth (`G2` and `G3`), Meta DFM never trusts a single first-draft trajectory:

1. **Parallel solutions and clones:** independent solver trajectories explore distinct approaches inside isolated working copies (at least two at `G2`, three at `G3`, unless explicitly exempted). Each parallel solution runs its own checks, computes its own SHA-256 evidence digest, and receives independent verification scores across the five review dimensions.
2. **Deterministic winner selection and contingency plans:** only solutions satisfying every rung, trust floor, mission-trace check, and zero-refutation rule are eligible. Eligibility takes precedence over raw score; the top-ranked eligible solution integrates into root and must pass verification again with matching content digests (`files_sha256`). Highest-scoring non-selected solutions are preserved in order as **`plan_b`** and **`plan_c`** for immediate fallback on regression or deployment failure.
3. **No majority voting — grounded refutation blocks acceptance:** review never averages away a concrete defect by majority vote. One well-grounded refutation backed by reproducible counterexamples, broken invariants, or observed failure traces halts acceptance until the flaw is fixed or the refutation itself is shown invalid against primary evidence.
4. **Portable learning note:** upon completion or halt, each piece emits a plain structured learning record capturing attempts, failure outputs, escalations, parallel solution rankings (`plan_b`, `plan_c`), unproven assumptions from `simulated proof`, and `limited_by` trust bottlenecks — feeding long-term reinforcement and method improvement without granting any agent silent write access to core governance.

---

## 7. Open Research Extensions for Multi-Agent Verification Graphs

Three hardening mechanisms remain active research extensions for integrating `C0–C4` across concurrent multi-agent graphs:

1. **Out-of-band tool-trace auditing before `C3–C4` promotion:** asynchronous verification inspects raw command transcripts, exit codes, and skipped/mocked branches out-of-band so self-reported summaries cannot mask silent failures before high trust is granted.
2. **Deterministic code-level `Reduce` with an explicit rejected set:** when aggregating parallel solutions, refutations, and findings across graph fan-outs, deduplication and ranking execute in deterministic code rather than unconstrained prose synthesis, retaining a `rejected` audit set explaining every discarded branch.
3. **Cardinality guard across phase transitions:** phase handoffs verify exact structural accounting (`len(expected) == len(received)` over dispatched tasks and prover checks) so dropped or timed-out parallel branches cannot be mistaken for silence or agreement.
