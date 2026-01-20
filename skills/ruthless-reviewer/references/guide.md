# Ruthless Reviewer Guide

Run a mediated, manual multi-agent review loop by ping-ponging feedback between reviewers and an implementer. The goal is convergence on superior outcomes, not consensus for its own sake.

---

## Core Principle

The loop is: Review -> Mediate -> Ping back -> Improve -> Repeat. The mediator enforces quality gates and convergence.

A reviewer is not a judge. A reviewer is a signal generator. The mediator owns synthesis and decisions.

---

## Severity Rubric

All reviewers must use this shared scale:

| Severity | Definition | Gate impact |
|----------|------------|-------------|
| **High** | Blocks shipping. Incorrect, unsafe, or fundamentally broken. | Must fix before close. |
| **Medium** | Should fix. Quality/maintainability issue but not blocking. | Fix or explicitly accept risk. |
| **Low** | Nice to have. Polish, style, minor improvements. | Fix if time permits. |

Reviewers label findings with H/M/L. The mediator uses these to enforce gates.

---

## Roles

- **Implementer**: the agent doing the work.
- **Mediator**: you. You set scope, reconcile disagreements, and decide what changes.
- **Reviewer(s)**: independent reviewers with distinct lenses.

Reviewer lenses you can assign:
- Ruthless reviewer (correctness and risks)
- Design/architecture reviewer (structure and tradeoffs)
- User-impact reviewer (UX, clarity, ergonomics)
- Test reviewer (coverage, edge cases)
- Red team reviewer (attack failure modes)

---

## Modes

### Multi-agent mode (2-4 reviewers)
The full ping-pong with multiple perspectives. Use when:
- High-stakes artifact
- Complex tradeoffs
- Need diverse lenses

### Two-agent mode (implementer + 1 reviewer)
The common case. One implementer, one ruthless reviewer. Use when:
- Iterating quickly
- Single dominant concern (e.g., correctness)
- Limited time

In two-agent mode:
- Skip "Disagreements" section (no one to disagree with)
- Convergence = reviewer has no new high-severity items
- The reviewer IS the ruthless reviewer

---

## Phase 1: Setup (Gate)

Define the review contract before any review happens.

### Inputs
- Artifact to review (code diff, plan, doc, spec)
- Scope boundaries (what is in/out)
- Quality bar (what "S-tier" means here)
- Time budget (rounds or minutes)

### Reviewer roster
- **Multi-agent mode**: Pick 2-4 reviewers with different lenses. Ensure at least one is a ruthless reviewer.
- **Two-agent mode**: One ruthless reviewer. That's it.

### Create a working log
Create `review-log.md` in the working directory. Use the template below so checks can be run.

**Gate to proceed:**
- Artifact and scope are stated
- Reviewers and lenses are selected
- Working log exists

---

## Phase 2: Round Loop

Each round has four steps: Review, Mediate, Implement, Ping-back.

### Step A: Review
Send each reviewer a brief with constraints and required output structure.

Reviewer brief template:
```
ROLE: [Reviewer lens]
TASK: Review the artifact for [goal].
SCOPE: [in scope] / [out of scope]
CONSTRAINTS:
- Be specific and actionable
- Avoid repeats unless you add new evidence
- If you disagree with another reviewer, say why

OUTPUT FORMAT:
1) Findings (ordered, severity-labeled)
2) Risks / missing considerations
3) Tests / verification
4) Suggested changes
5) Confidence (low/medium/high)
```

### Step B: Mediate
Synthesize all responses into a single decision artifact. This is where convergence happens.

Mediator synthesis template:
```
### Round N Synthesis

**Consensus:**
- ...

**Disagreements:** (skip in two-agent mode)
- Reviewer A vs B: [topic]
  - Decision: [what we do and why]

**Actions:**
- [ ] Action 1 (owner: implementer)
- [ ] Action 2

**Open Questions:**
- ...

**Gate Status:**
- New high-severity items? [yes/no]
- All actions addressed? [yes/no]
- Ready to close? [yes/no]
```

### Step C: Implement
Before ping-back, implement the agreed changes. Don't review stale artifacts.

- Implementer addresses actions from synthesis
- Log what changed: `### Changes (Round N)`
- Gate: Changes logged before ping-back

### Step D: Ping-back
Send targeted follow-ups only. Ask for deltas, not repeats.

Ping-back prompt:
```
Here is the synthesis. Respond with:
- Any critical misses
- Any disagreement with the decision
- Any new high-severity issue only
No repeats.
```

**Round gate:**
- Each reviewer response uses H/M/L severity labels
- Synthesis includes Consensus, Actions, Gate Status
- Changes logged before ping-back (rounds > 1)

---

## Phase 3: Convergence Gate

Stop when the loop converges. Use one of these stop conditions:
- Two consecutive rounds with no new high-severity items
- All actions addressed and reviewers have no new deltas
- Time budget reached and risk is explicitly accepted

**Close-out artifact:** A final synthesis with decisions and next steps.

---

## Drop Criteria (Reviewer Pruning)

Drop a reviewer when:
- They repeat prior points without new evidence
- Their feedback is consistently non-actionable
- They fail to follow the format twice
- They lower the quality bar (overly lenient)

If dropped, document why in the log.

---

## Eval Checks

These checks are runnable if you keep the log structure.

| # | Check | Command | Pass |
|---|-------|---------|------|
| 1 | Log exists | `test -f review-log.md` | File exists |
| 2 | Each round has synthesis | `grep -c "^### Round .* Synthesis" review-log.md` | Count ≥ 1 |
| 3 | Synthesis has required sections | `grep -E "^\*\*(Consensus|Actions|Gate Status):" review-log.md` | All 3 present |
| 4 | Actions are tracked | `grep -c "^- \[ \]" review-log.md` | Count matches actions |
| 5 | Changes logged (if round > 1) | `grep -c "^### Changes" review-log.md` | Count = rounds - 1 |

```bash
# Quick check script
echo "1) Log exists:"
test -f review-log.md && echo "PASS" || echo "FAIL"

echo "2) Synthesis blocks:"
grep -c "^### Round .* Synthesis" review-log.md || echo "0"

echo "3) Required sections:"
grep -E "^\*\*(Consensus|Actions|Gate Status):" review-log.md | wc -l

echo "4) Open actions:"
grep -c "^- \[ \]" review-log.md || echo "0"

echo "5) Changes logged:"
grep -c "^### Changes" review-log.md || echo "0"
```

If any check fails, fix the log structure before proceeding.

---

## Failure Modes

| Symptom | Cause | Fix |
|---|---|---|
| Reviewers keep repeating | No delta-only constraint | Add ping-back rule, enforce no repeats |
| Convergence stalls | Disagreements not mediated | Force explicit decisions in synthesis |
| Low signal feedback | Wrong reviewer lens | Replace or re-brief reviewer |
| Too many issues, no action | Missing prioritization | Rank by severity and cut scope |
| Reviewer contradicts themselves | No evidence requirement | Ask for evidence or drop |

---

## Working Log Template

Create `review-log.md` and append per round.

```
# Review Log

## Artifact
- Description: ...
- Scope: ...
- Quality bar: ...

## Reviewers
- Reviewer A: [lens]
(two-agent mode: just one reviewer)

---

## Round 1

### Reviewer A
[Paste response with H/M/L severity labels]

### Round 1 Synthesis

**Consensus:**
- ...

**Disagreements:** (skip in two-agent mode)
- ...

**Actions:**
- [ ] ...

**Open Questions:**
- ...

**Gate Status:**
- New high-severity items? [yes/no]
- All actions addressed? [yes/no]
- Ready to close? [yes/no]

---

## Round 2

### Changes (Round 1)
- [What was implemented from Round 1 actions]

### Reviewer A
[Paste delta-only response]

### Round 2 Synthesis
...
```

---

## Notes on Structure Emergence

Do not over-prescribe the format until you hit friction.
- If reviewers miss key areas, tighten the template.
- If output feels bloated, remove sections.
- If convergence is slow, add stricter stop conditions.

The structure should emerge from failure modes, not from theory.
