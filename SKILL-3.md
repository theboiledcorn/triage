---
name: structural-triage-skill
description: Six-gate structural triage with Phase 0 prework, Phase 2 gate failure classification, severity, output format, and anti-patterns. Independent of protocol, exploit class, or code patterns.
---

# Structural Triage Skill

A framework for evaluating smart contract vulnerability submissions. Independent of protocol, exploit class, or specific code patterns. Based on structural questions about state ownership, design promises, causal control, custody, and deployment scope.

---

## Phase 0 — Prework (Do Not Skip)

**Read the code. Read the bounty scope. Do not assume.**

Before touching any finding:

1. Read the actual source file referenced. Not the snippet in the report — the file.
2. Check the bounty program's assets table. If the contract is not listed, the component is out of scope. Stop.
3. Check the program's out-of-scope rules. Common exclusions: privileged address attacks, best-practice critiques, feature requests, contracts that are documented as unused.
4. Identify what the contract actually stores and promises. Not what the report claims it should do.

Output nothing yet. This is internal context.

---

## Phase 1 — Decision Gates

Run every finding through all six gates. If any gate fails, the finding is Invalid or Informational (see Phase 2 for which).

### Gate 1 — Canonical State and Ownership

**Ask:** Is the reported harm in the contract's own storage, or in a derived view, off-chain event, or external contract?

- If the asset is unowned, intentionally transient, or collectible by anyone: no ownership violation.
- If the harm exists only off-chain (event data, UI, external accounting): not a vulnerability in this contract.
- If the harm is in a different contract that the finding doesn't target: narrow the scope.

**Fail:** Harm outside canonical state, or in an intentionally unowned asset.

### Gate 2 — Scope of Promise and Design Intent

**Ask:** Does this contract actually promise to prevent the reported outcome?

- Read the code, comments, and NatSpec. Look for documentation that the behavior is intended.
- If the code has a caller-supplied execution capability (arbitrary route bytes, user-chosen hook address, etc.) and the "vulnerability" is that the caller chose badly: no broken promise.
- If the contract explicitly documents the behavior as a known trade-off: no broken promise.
- If the behavior is a necessary consequence of the contract's design: no broken promise unless the contract claims otherwise.

**Fail:** Behavior is documented, advertised, or a necessary consequence of the design. No promise was made.

### Gate 3 — Strongest Invariant Trade-off

**Ask:** Is the observed behavior an unavoidable by-product of a stronger safety property?

- Example: a reentrancy lock that temporarily blocks calls is not a DoS vulnerability.
- Do not use this gate to excuse custody failures just because "the victim could have avoided it." That belongs under Gate 4 or severity.

**Fail:** The behavior is a necessary protective trade-off.

### Gate 4 — Causal Control and Victim Participation

**Ask:** Who supplies the decisive inputs that produce the harm? Can an external actor force the outcome without the victim's consent?

- List every input that influences the outcome.
- If the victim supplies ALL of them and could have chosen different values to avoid the harm without giving up the core functionality they intended: self-inflicted. Fail.
- If an external actor can alter the outcome without the victim's cooperation (front-running, manipulating shared state, permissionless call): control is not solely with the victim. Pass.

**Fail:** Victim exclusively controls all decisive inputs and can trivially avoid the harm.

### Gate 5 — Custody and Reversibility

**Ask:** Were the victim's assets in the contract's temporary custody? Does the contract fail to return them?

- If the assets are residual balances from prior operations with no ownership record: abandoned dust. Fail.
- If the contract provides any path for the user to reclaim stranded funds within the same transaction: no loss. Fail.
- If the contract never had a duty to return the value: Fail.

**Pass:** User funds in custody are lost or stranded with no reclaim path.

### Gate 6 — Configuration and Deployment Assumptions

**Ask:** Does the vulnerability require an absurd, extreme, or warned-against configuration?

- If the issue relies on the owner setting a parameter to an extreme value that no rational deployment would use: Fail.
- If the issue requires configuring 256 modules, duplicate validators, unsupported token types, or a chain without a required hard fork: Fail.
- If the issue only affects a feature that is not deployed (not in assets table): Fail.

**Pass:** The issue works under normal, supported deployment.

---

## Phase 2 — Gate Failure Classification

If a gate failed, determine whether the finding is Invalid or Informational.

**Invalid** if:
- Component is not in the assets table (unused / unintegrated)
- Requires a privileged address to act maliciously (out of bounty scope)
- Victim controls all inputs and self-inflicted the harm
- False positive (misunderstands the code or architecture)
- Duplicate of another finding with the same root cause and impact

**Informational** if:
- Behavior is documented and working as designed (but worth noting)
- Requires an extreme precondition (chain fork, signer signing two same-second quotes)
- Theoretical edge case with no practical path
- Code quality observation (missing validation, missing gap, best-practice deviation) with no loss path
- Feature request or design observation

---

## Phase 3 — Severity Classification (If All Gates Pass)

Only classify severity after all six gates pass. Severity is based on the nature of the harm, not the mechanism.

- **Critical:** Irreversible loss of the user's principal that the victim could not reasonably prevent while using the protocol as intended. The attacker does not need the victim's cooperation.
- **High:** Meaningful loss of user funds or a clear breach of a custody/safety invariant, even if the victim could have reduced exposure with extreme care.
- **Medium:** Externally caused harm with no direct theft of principal. Typically griefing (transaction failure, gas waste) that the victim cannot trivially avoid without sacrificing functionality.
- **Low:** The contract fails to deliver promised functionality (clean revert, incorrect event, tiny accounting mismatch) but no meaningful value is lost.
- **Informational:** The observation is technically correct but relies on out-of-scope preconditions, unsupported configurations, or missing features.

Custody-failure sub-rule: If the contract takes custody of an asset for an operation and fails to return the unused portion, that unreturned portion is stolen principal. Severity depends on magnitude and preventability.

---

## Phase 4 — Output Format

One summary per finding. No em dashes. Backticks on function names and code references.

```
# [ID]: [descriptive title]

**Verdict**: [Critical/High/Medium/Low/Informational/Invalid]

**Why?** [2-5 sentences. Independent reasoning grounded in the code. State what the code does, who controls the inputs, why the gate passed or failed.]

**Caveats**: [Any conditions, edge cases, or context that limits the finding. Empty if none.]
```

---

## Anti-Patterns

1. **Assuming deployment without checking the assets table.** If it's not in the assets table, it's not in scope. Stop there.
2. **Calling things duplicates without independent analysis.** Every finding gets its own gate evaluation.
3. **Confusing "the attacker does X" with "the attacker can force X."** Check Gate 4: does the attacker have causal control, or does the victim?
4. **Excusing custody failures with "the victim could have avoided it."** That's Gate 4 analysis, not a shortcut.
5. **Parroting the reviewer's reasoning.** Write your own analysis. Verify everything against the code.
6. **Batch-processing without per-finding verification.** Go one by one. Each finding has its own code path, own input sources, own deployment status.
7. **Forgetting backticks on function names.** Format matters for readability.
