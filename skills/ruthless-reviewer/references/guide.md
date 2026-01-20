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
- **Mediator**: you (the human). You set scope, reconcile disagreements, and decide what changes.
- **Reviewer(s)**: independent reviewers with distinct lenses.

**Critical:** The implementer must STOP and escalate to the mediator - not proceed autonomously - when design decisions arise.

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

## Orchestration

Choose how you run the loop:

### Manual log (copy/paste)
Use `review-log.md` and paste reviewer responses each round. This is the default and is fully checkable with the eval checks.

### MCP thread (Codex)
If you have Codex MCP available (`mcp__codex__codex` and `mcp__codex__codex-reply` tools), you can run the reviewer through a single MCP thread and avoid manual copy/paste. Create `review-session.md` and record the thread ID plus round summaries so the loop is auditable.

**Note:** MCP mode is optimized for two-agent mode (one reviewer). For multi-agent reviews, use manual mode or run multiple MCP threads (one per reviewer lens).

**Prerequisite:** Codex MCP server configured. See `codex mcp-server --help` for setup.

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

### Create a session record
Pick one:
- **Manual log:** Create `review-log.md` in the working directory. Use the template below so checks can be run.
- **MCP thread:** Create `review-session.md` and record the thread ID. Use the template below.

**Gate to proceed:**
- Artifact and scope are stated
- Reviewers and lenses are selected
- `review-log.md` or `review-session.md` exists

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

### MCP option: Reviewer thread
If using Codex MCP, start the reviewer in a single thread and reuse it across rounds:

Round 1 (start new thread):
```
mcp__codex__codex(prompt="You are the ruthless reviewer... [artifact path] [scope] [quality bar] [output format]", cwd="...") -> threadId
```

Round N (reuse thread):
```
mcp__codex__codex-reply(threadId="...", prompt="Here is the synthesis + changes. Provide only deltas/new high severity items.")
```

Record `threadId` and round summaries in `review-session.md`.

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

**Decision points:** none this round.
(or full DECISION POINT template if triggers fired)

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

**STOP for escalation** if any action involves:

| Trigger | Action |
|---------|--------|
| Design tradeoff | Present options A/B with pros/cons |
| "By design" response | Don't dismiss - escalate to mediator |
| Scope change | Confirm before proceeding |
| Repeated issue (2+ rounds) | May indicate deeper problem |
| Accepting limitation | Mediator decides, not implementer |
| Architectural choice | Affects overall structure |

**When triggers fire** - STOP and present decision:

**Option 1: AskUserQuestion tool** (Claude Code)
```
AskUserQuestion({
  questions: [{
    question: "[Issue]? Stake: [what gets worse if wrong]",
    header: "Design",
    options: [
      { label: "Option A (Recommended)", description: "[approach] - Pro: X, Con: Y" },
      { label: "Option B", description: "[approach] - Pro: X, Con: Y" }
    ],
    multiSelect: false
  }]
})
```

**Option 2: Text template** (portable fallback)
```
DECISION POINT: [issue]
Stake: [what gets worse if we choose wrong]

Option A: [approach]
- Pro: ...
- Con: ...

Option B: [approach]
- Pro: ...
- Con: ...

Recommendation: [A/B] because [reasoning]

Your call?
```

Wait for explicit response before proceeding. Log: `Approved: [decision] by [user]`

**When no triggers fire** - state explicitly in synthesis:
```
Decision points: none this round.
```

This creates an auditable trace that the agent checked. Silence cannot masquerade as compliance.

**Discretion expected.** The trigger list is intentionally tight. Don't escalate every minor choice - that defeats the purpose too. Escalate design tradeoffs that shape the outcome.

### Step D: Ping-back
Send targeted follow-ups only. Ask for deltas, not repeats.

Ping-back prompt:
```
Here is the synthesis. Respond with:
- Any critical misses (H/M/L labeled)
- Any disagreement with the decision
- Any new high-severity issue only
No repeats. Use H/M/L severity labels.
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

Choose the checks based on orchestration mode.

**Note:** Count-based checks (rounds, actions, changes) print values for human verification. Pass/fail is manual - compare counts against expected values.

### Manual log checks (`review-log.md`)

| # | Check | Command | Pass |
|---|-------|---------|------|
| 1 | Log exists | `test -f review-log.md` | File exists |
| 2 | Each round has synthesis | `grep -c "^### Round .* Synthesis" review-log.md` | Count ≥ 1 |
| 3 | Synthesis has required sections | `grep -E "^\*\*(Consensus|Actions|Gate Status):" review-log.md` | All 3 present per round |
| 4 | H/M/L findings captured | `grep -qE "^- [HML]:|High:|Medium:|Low:" review-log.md` | Exit 0 |
| 5 | Gate Status per round | `grep -c "^\*\*Gate Status:" review-log.md` | Count = rounds |
| 6 | Changes logged (if round > 1) | `grep -c "^### Changes" review-log.md` | Count = rounds - 1 |
| 7 | Decision points addressed | `grep -qE "Decision points:|DECISION POINT:" review-log.md` | Exit 0 |

```bash
# Quick check script
echo "1) Log exists:"
test -f review-log.md && echo "PASS" || echo "FAIL"

echo "2) Synthesis blocks:"
grep -c "^### Round .* Synthesis" review-log.md || echo "0"

echo "3) Required sections:"
grep -E "^\*\*(Consensus|Actions|Gate Status):" review-log.md | wc -l

echo "4) H/M/L findings:"
grep -qE "^- [HML]:|High:|Medium:|Low:" review-log.md && echo "PASS" || echo "FAIL"

echo "5) Gate Status per round:"
grep -c "^\*\*Gate Status:" review-log.md || echo "0"

echo "6) Changes logged:"
grep -c "^### Changes" review-log.md || echo "0"

echo "7) Decision points addressed:"
grep -qE "Decision points:|DECISION POINT:" review-log.md && echo "PASS" || echo "FAIL"
```

### MCP session checks (`review-session.md`)

| # | Check | Command | Pass |
|---|-------|---------|------|
| 1 | Session exists | `test -f review-session.md` | File exists |
| 2 | Thread ID recorded | `grep -q "^Thread ID:" review-session.md` | Exit 0 |
| 3 | At least one round | `grep -c "^## Round" review-session.md` | Count ≥ 1 |
| 4 | Rounds have gate status | `grep -c "^\*\*Gate Status:" review-session.md` | Count = rounds |
| 5 | H/M/L findings captured | `grep -qE "^- [HML]:" review-session.md` | Exit 0 |
| 6 | Changes logged (if round > 1) | `grep -c "^### Changes" review-session.md` | Count = rounds - 1 |
| 7 | Decision points addressed | `grep -qE "Decision points:|DECISION POINT:" review-session.md` | Exit 0 |
| 8 | Final synthesis exists | `grep -q "^## Final Synthesis" review-session.md` | Exit 0 |

```bash
# Quick check script (MCP)
echo "1) Session exists:"
test -f review-session.md && echo "PASS" || echo "FAIL"

echo "2) Thread ID:"
grep -q "^Thread ID:" review-session.md && echo "PASS" || echo "FAIL"

echo "3) Round count:"
grep -c "^## Round" review-session.md || echo "0"

echo "4) Gate status per round:"
grep -c "^\*\*Gate Status:" review-session.md || echo "0"

echo "5) H/M/L findings:"
grep -qE "^- [HML]:" review-session.md && echo "PASS" || echo "FAIL"

echo "6) Changes logged:"
grep -c "^### Changes" review-session.md || echo "0"

echo "7) Decision points addressed:"
grep -qE "Decision points:|DECISION POINT:" review-session.md && echo "PASS" || echo "FAIL"

echo "8) Final synthesis:"
grep -q "^## Final Synthesis" review-session.md && echo "PASS" || echo "FAIL"
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
| MCP session lost | Thread ID not recorded | Record in `review-session.md` and restate context |
| Implementer steamrolls decisions | Not escalating tradeoffs | Review escalation triggers, re-examine "by design" calls |
| "By design" used defensively | Avoiding work vs genuine tradeoff | Mediator must approve all "by design" responses |
| Fast convergence, wrong outcome | Implementer and reviewer aligned but wrong | Human mediator validates key decisions, not just pass/fail |

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
[Paste delta-only response with H/M/L labels]

### Round 2 Synthesis
...
```

---

## MCP Session Template

Create `review-session.md` if using Codex MCP. Must capture same gates as manual mode.

```
# Review Session

## Artifact
- Description: ...
- Scope: ...
- Quality bar: ...

## Reviewers
- Reviewer A: [lens] (Codex MCP)

Thread ID: [from mcp__codex__codex response]

---

## Round 1

### Reviewer Findings
[Paste or summarize Codex response - must include H/M/L labels]
- H: ...
- M: ...
- L: ...

### Round 1 Synthesis

**Consensus:**
- ...

**Disagreements:** (skip in two-agent mode)
- ...

**Actions:**
- [ ] ...

**Decision points:** none this round.
(or full DECISION POINT template if triggers fired, with "Approved: [decision] by [user]")

**Open Questions:**
- ...

**Gate Status:**
- New high-severity items? [yes/no]
- All actions addressed? [yes/no]
- Ready to close? [yes/no]

---

## Round 2

### Changes (Round 1)
- [What was implemented]

### Reviewer Findings
[Paste or summarize mcp__codex__codex-reply response with H/M/L labels]
- H: ...
- M: ...
- L: ...

### Round 2 Synthesis

**Consensus:**
- ...

**Disagreements:** (skip in two-agent mode)
- ...

**Actions:**
- [ ] ...

**Decision points:** none this round.
- [ ] ...

**Open Questions:**
- ...

**Gate Status:**
- New high-severity items? [yes/no]
- All actions addressed? [yes/no]
- Ready to close? [yes/no]

---

## Final Synthesis
- Decision: ...
- Risks accepted: ...
- Next steps: ...
```

---

## Exemplars

Study these before running a review:

- **skill-crafting review** (thread `019bda94-d0c9-7c23-aae0-d4d933a2547d`) - 4-round MCP session. Demonstrates severity progression H→M→L→clear, mid-flight fixes, convergence.

**In progress:**
- Self-review of this skill (recursive validation)

---

## Notes on Structure Emergence

Do not over-prescribe the format until you hit friction.
- If reviewers miss key areas, tighten the template.
- If output feels bloated, remove sections.
- If convergence is slow, add stricter stop conditions.

The structure should emerge from failure modes, not from theory.
