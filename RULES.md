For each vulnerability, produce the following structured deliverable UNVARNISHED.
Fill every section with as much detail as possible.
Commit to a conclusion based on available evidence.
Explicitly identify uncertainty and missing evidence — do not fabricate confidence.
If source code is unavailable for any sub-section, write
CODE REQUIRED: [exact filename + function name] and halt that section.

─────────────────────────────────────────────
1. FEYNMAN (SIMPLE) EXPLANATION
─────────────────────────────────────────────
One paragraph. Lead with the impact, then explain the root cause in plain language.
Define every technical term on first use. No jargon without a definition.

─────────────────────────────────────────────
2. ROOT CAUSE
─────────────────────────────────────────────
Category:              [e.g., Missing authorization check]
Trust boundary violated: [e.g., User-controlled input influences privileged state update]
Broken invariant:      [e.g., Only approved operator should modify position ownership]

─────────────────────────────────────────────
3. PREREQUISITES & ATTACK SCENARIO
─────────────────────────────────────────────
Attacker position:
Required configuration:
User interaction required:

Step-by-step attack narrative — from nothing to impact:
  Step 1: ...
  Step 2: ...
  Final:  Attacker achieves X

─────────────────────────────────────────────
4. CODE PATH WALK-THROUGH
─────────────────────────────────────────────
Entry point(s): [function name, visibility, file:line]

Complete call chain — for every hop annotate all of the following:

  caller() [File.sol:L]
    State read:               [variable name or NONE]
    State write:              [variable name or NONE]
    External call:            [target contract + function or NONE]
    Privilege boundary:       [crossed / not crossed]
    User-controlled input:    yes / no

    → intermediate() [File.sol:L]
      State read:             ...
      State write:            ...
      External call:          ...
      Privilege boundary:     ...
      User-controlled input:  ...

      → vulnerableFn() [File.sol:L]  ← sink
        State read:           ...
        State write:          ...
        External call:        ...
        Privilege boundary:   ...
        User-controlled input: yes / no

Sink classification:
  Type: [unauthorized state write / token transfer / external call /
         delegatecall / signature verification / oracle consumption /
         arithmetic operation / access control decision /
         liquidation logic / accounting update]

Input validation along the path:
  What is checked:
  What is not checked:
  Sanitizers or modifiers missing or flawed:

Trace diagram:
  entryFn() → intermediateFn() → vulnerableFn(user_input) → [sink]

─────────────────────────────────────────────
5. ASSUMPTIONS THE VULNERABILITY RELIES ON
─────────────────────────────────────────────
For each precondition the attacker must meet:

  Assumption:
  Evidence supporting:
  Evidence contradicting:
  Likelihood: X%

─────────────────────────────────────────────
6. KILL CONDITIONS
─────────────────────────────────────────────
This finding is invalid if any of the following hold:
  - Condition 1
  - Condition 2
  - Condition 3

For each condition: does it hold in this codebase? [yes / no / unknown — CODE REQUIRED]

─────────────────────────────────────────────
7. PoC / REPRO EVIDENCE  (DO NOT WRITE WORKING EXPLOIT)
─────────────────────────────────────────────
What the PoC proves:
Whether it works reliably:
Reproduction steps: exact environment, commands, expected vs actual output.

─────────────────────────────────────────────
8. EXPLOITABILITY CONFIDENCE
─────────────────────────────────────────────
Technical validity:       X%  [is the code path real and traceable?]
Practical exploitability: Y%  [can an attacker realistically trigger it?]
Impact confidence:        Z%  [is the stated impact accurate if triggered?]

Evidence FOR exploitability:
Evidence AGAINST (what would make this a false positive):
What would raise confidence: [specific code, config, or test result needed]

─────────────────────────────────────────────
9. IMPACT ANALYSIS
─────────────────────────────────────────────
Confidentiality: [what becomes readable / exposed]
Integrity:       [what state becomes corruptible / controllable]
Availability:    [what can be locked, bricked, or disrupted]

Data exposed or controllable:
Lateral movement or chain potential:
Worst-case scenario if exploited:

─────────────────────────────────────────────
10. ECONOMIC FEASIBILITY
─────────────────────────────────────────────
Estimated attacker cost:
Estimated attacker profit:
Capital requirements:
MEV / front-run competition:
Would a rational attacker execute? Why or why not?

─────────────────────────────────────────────
11. REMEDIATION & WORKAROUNDS
─────────────────────────────────────────────
Proposed patch: [specific code diff or config change]
Temporary mitigation:
Effort to fix: [story points or hours]
Regression risk: [what else might break, which tests to run]

─────────────────────────────────────────────
12. TRIAGE DECISION
─────────────────────────────────────────────
Recommended action: Emergency patch / Schedule / Defer / Close (not applicable)

Decision matrix:
  Exploitability (1–5): X — basis: [code path confirmed / requires specific state / etc.]
  Impact (1–5):         X — basis: [concrete harm if fired]
  Score: Exploitability × Impact = Z

Reasoning tied to business context:
If deferred: re-evaluate by [specific date or trigger condition]

─────────────────────────────────────────────
13. EVIDENCE
─────────────────────────────────────────────
Direct code evidence:    [File.sol:Lx–Ly]
Supporting code:         [File.sol:Lx–Ly]
Tests reviewed:          [test file names or NONE]
Deployment assumptions:  [e.g., proxy owner = deployer, pausable = false]
External references:     [SWC ID, Solodit pattern, Rekt incident — only if directly analogous]
