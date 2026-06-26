---
name: structural-triage-framework
description: Six-gate structural triage for smart contract vulnerability submissions. Evaluates state ownership, design promises, invariant trade-offs, causal control, custody, and deployment configuration.
---

# Structural Triage Framework

Evaluate any smart-contract vulnerability submission without reference to specific protocols, functions, or exploit classes. Ask structural questions about state ownership, design promises, causal control, custody, and harm.

---

## Decision Gates

### Gate 1 — Canonical State and Ownership

**Question:** Is the reported harm in the authoritative state of the contract under review, or only in a derived view, external record, or user-controlled input?

- If the asset or state is explicitly treated as unowned, collectible by anyone, or intentionally transient, there is no ownership violation.
- If the harm exists only outside the contract's own storage (e.g., an off-chain event, a UI misrepresentation), it is not a vulnerability in this contract.

**Verdict:**
Harm outside the contract's canonical state or in an intentionally unowned asset → **Invalid**.

---

### Gate 2 — Scope of Promise and Design Intent

**Question:** Does the contract actually promise to prevent the reported outcome?

- Look at code structure, comments, or guarded alternative functions that show the behavior is intended.
- If the outcome is merely a consequence of a documented, caller-supplied execution capability (e.g., arbitrary route bytes), there is no broken promise.
- The promise must be inferred from the contract's own logic, not from user expectations.

**Verdict:**
Behavior is an advertised capability or no such guarantee exists → **Invalid** or **Informational**.
A real design promise exists and is broken → proceed.

---

### Gate 3 — Strongest Invariant Trade-off

**Question:** Is the observed behavior a violation of a core invariant, or is it the unavoidable cost of preserving an even stronger safety property?

- This gate applies only when the behavior is a direct, necessary by-product of a higher-priority protection (e.g., a reentrancy lock that temporarily blocks calls is not a DoS vulnerability).
- Do not use this gate to excuse custody failures just because the victim could have avoided the loss with different parameters.

**Verdict:**
Behavior is a necessary protective trade-off → **Invalid**.
Otherwise, proceed.

---

### Gate 4 — Causal Control and Victim Participation

**Question:** Who supplies the decisive inputs that produce the harm, and can an external actor force the outcome without the victim's consent?

- List every input that influences the outcome. If they are all supplied by the victim and the victim could have chosen different values to avoid the harm without giving up the core functionality they intended to use, the harm is self-inflicted.
- If an external actor can alter the outcome without the victim's cooperation (e.g., by front-running a public transaction, by manipulating shared state), control is not solely with the victim.

**Verdict:**
Victim exclusively controls all parameters and can trivially avoid the harm → **Invalid**.
External actor can force the outcome without consent → proceed.

---

### Gate 5 — Custody and Reversibility

**Question:** Were the assets in the contract's temporary custody, and does the contract fail to return them?

- If the assets are residual balances from prior operations with no ownership record, they are abandoned dust.
- If the assets are user funds temporarily held for an ongoing operation, check whether the contract guarantees their return. If the operation can complete while leaving unspent user funds behind with no same-transaction reclaim path for the user, that is a custody failure.
- If the contract provides any path for the user to reclaim the stranded funds within the same transaction, there is no loss.

**Verdict:**
Contract never had a duty to return the value → **Invalid**.
User funds in custody are lost or stranded with no reclaim path → proceed.

---

### Gate 6 — Configuration and Deployment Assumptions

**Question:** Does the vulnerability require an absurd or explicitly warned-against deployer configuration?

- If the issue relies on the owner setting a parameter to an extreme value that the documentation already warns against, it is an admin-error, not a logic flaw.
- The contract's normal, supported deployment should be the baseline.

**Verdict:**
Depends on unreasonable configuration → **Invalid** (or Informational).
Works under normal deployment → proceed.

---

## Severity Classification

Only after passing all six gates do we assess severity.

- **Critical** — Irreversible loss of the user's principal that the victim could not reasonably prevent while using the protocol as intended. The attacker does not need the victim's cooperation.
- **High** — Meaningful loss of user funds or a clear breach of a custody/safety invariant, even if the victim could have reduced exposure with extreme care.
- **Medium** — Externally caused harm with no direct theft of principal. Typically griefing (transaction failure, gas waste) that the victim cannot trivially avoid without sacrificing functionality.
- **Low** — The contract fails to deliver promised functionality (clean revert, incorrect event, tiny accounting mismatch) but no meaningful value is lost.
- **Informational** — The observation is technically correct but relies on out-of-scope preconditions, unsupported configurations, or missing features. No valid vulnerability under the program's rules.

**Custody-failure sub-rule:** If the contract takes custody of an asset for a specific operation and fails to return the unused portion, that unreturned portion is stolen principal.

---

## Duplicate Consolidation

If the finding is valid, check whether the same root cause (same broken invariant) and same impact class have already been reported. If so, mark as duplicate and reference the primary.

---

## Summary Decision Flow

1. Harm in the contract's own canonical state? (and not intentionally unowned)
2. Contract actually promises to prevent this outcome?
3. Not an unavoidable protective trade-off?
4. Victim does not exclusively control all decisive inputs?
5. User funds in temporary custody that cannot be reclaimed?
6. Works under normal supported deployment?

If any answer is "no" → **Invalid** (or Informational).
If all are "yes" → **Valid**. Then classify severity.
