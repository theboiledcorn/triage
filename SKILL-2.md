---
name: triage-skill-refined
description: Four-gate structural triage asking whose state got corrupted, what was promised, who controlled the chain of events, and whether the fix would break the system's real job.
---

# The Triage Skill (Refined)

This is not a checklist. It is a reasoning discipline. It works for any system — smart contracts, infrastructure, financial products — because it asks only structural questions about state, promises, control, and intervention effects.

---

## The One-Breath Summary

> Whose state got corrupted?
> Was that corruption ever supposed to be impossible?
> Who controlled the chain of events?
> Would the natural fix break the system's real job?

If you can answer those four questions, you know whether a finding is valid and how to frame it.

---

## The Four Structural Gates

Apply these in order. Each gate that fails terminates the analysis with a verdict.

### 1. Canonical State and Ownership

**What to ask:**
- Is the reported damage in the component's own authoritative ledger, or only in a derived view, external mock, or user-controlled record?
- Who owns the truth of that data? If the component is not the owner, the corruption is not its responsibility.

**Verdict:**
- Harm confined outside the component's canonical state → **not a vulnerability of that component**.
- Harm inside the component's authoritative state → continue.

---

### 2. Scope of Promise

**What to ask:**
- Does the system — anywhere in its documentation, comments, or guarded logic — commit to preventing this exact outcome?
- Is the reported behaviour an intended consequence of a feature (e.g., a user-facing capability), or a failure of one?

**Verdict:**
- The system never promised what the finding claims → **informational / out of scope**.
- The finding describes an intended capability used in an unintended way, but the capability itself is deliberate → **informational, user-choice issue**.
- The system explicitly or implicitly promises the opposite → continue.

---

### 3. Causal Control

**What to ask:**
- Trace the minimal sequence that produces the harm. Who supplies the decisive inputs?
- Must the victim actively cooperate after the malicious condition already exists? (Install a malicious component, approve it, sign a message, call a function.)
- Does the system present itself as protecting users from this class of interaction? If yes, then "they clicked it" may not be a sufficient dismissal.

**Verdict:**
- The victim's own voluntary action is the sole trigger, and the system never claimed to guard against that → **self-inflicted / informational**.
- An external actor can force the outcome without victim consent → continue.

---

### 4. Intervention Impact

**What to ask:**
- What is the most obvious fix that would prevent the reported state?
- If that fix were applied, would it weaken a higher-priority property of the system (e.g., liveness, availability, permissionlessness)?
- In other words, would fixing it break the system's real job?

**Verdict:**
- The fix would degrade a stronger invariant (e.g., allow a malicious extension to permanently lock a user's assets) → **the current behaviour is a protective trade-off, not a vulnerability**.
- The fix does not harm any higher-priority property → continue.

---

## After Validity: Severity, Not Existence

Only after a finding passes all four gates do you classify its severity. Use impact, not cleverness.

- **Critical:** Irreversible principal loss that the victim cannot reasonably prevent.
- **High:** Meaningful loss of value or custody, even if some care could reduce exposure.
- **Medium:** Externally caused harm or griefing without direct theft.
- **Low:** Broken feature, revert, event mismatch, or trivial accounting discrepancy.
- **Informational:** Passes some gates but fails on scope, self-infliction, or trade-off.

**Incentive rationality is a severity factor, not a validity filter.** A dumb, unprofitable attack can still be a real vulnerability; it just gets a lower severity.

---

## The Self-Inflicted Trap

"Self-inflicted" is not a magic dismissal. Ask the harder question: **Was the system designed to protect users from this mistake?**

- In a permissionless, extensible system where users freely add third-party components, the system does not promise to save users from their own component choices.
- In a system that presents itself as a safe wrapper around a dangerous operation, "they clicked it" may not be an adequate defence.

Always anchor "self-inflicted" in the system's explicit trust model.

---

## The Mental Model in Practice

When you receive a finding, do not start by trying to classify the vulnerability type. Start with:

1. **State:** Where is the damage? Is it in my component's truth, or somewhere else?
2. **Promise:** Did I ever say I'd prevent that?
3. **Control:** Whose hand caused it? Did they have to trust the attacker first?
4. **Intervention:** If I change this, does the whole building fall down?

By the time you answer those four, the verdict is usually obvious.

---

## The Skill in a Single Sentence (Abstract)

> Locate the authoritative state, identify what the system actually guarantees, trace who controlled the outcome, and test whether the most natural fix would sacrifice the system's primary mission — in that order, without ever relying on vulnerability patterns or domain-specific jargon.
