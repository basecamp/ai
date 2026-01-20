# Skill Crafting Guide

Create effective agent skills through co-development. Skills improve through use, not through thought.

---

## Core Principle: The Co-Development Flywheel

You can't study exemplars that don't exist. Instead, skill and output improve together:

```
Skill v0 + Target 1
       ↓
   Co-develop (skill ↔ output)
       ↓
   Output 1 graduates → Exemplar 1
       ↓
Skill v1 + Target 2
       ↓
   Co-develop (skill ↔ output)
       ↓
   Output 2 graduates → Exemplar 2
       ↓
   ... continue until mature
```

Each flywheel turn:
- Skill handles more edge cases
- Eval loop catches more failures
- Failure modes table grows from real friction
- Structure emerges where needed

---

## Phase 1: Bootstrap

No exemplars yet. Take a stab. Be wrong.

### Define the problem
- What task does this skill perform?
- What does "good" look like (even if vague)?
- Who will use this skill?

### Draft v0
Create `skills/SKILL_NAME/SKILL.md`:

```yaml
---
name: skill-name
description: One line describing what the skill does
triggers:
  - phrase that should activate this skill
  - another trigger phrase
---

# skill-name

Open `@references/guide.md` and follow it. Do not proceed without it.

[Brief description of what the skill produces]
```

Create `skills/SKILL_NAME/references/guide.md`:
- Rough process outline
- Initial templates (if applicable)
- Placeholder for eval checks
- Empty failure modes table

**v0 will be wrong. That's the point.**

---

## Phase 2: Co-Develop

This is the engine. Skill and output improve in tandem.

### Pick Target 1
- Real project, not a toy
- Representative of the skill's intended use
- You care about the output quality

### Work the target
Execute the skill on Target 1. As you work:

1. **Notice friction** - Where does the skill fail to guide you?
2. **Notice gaps** - What decisions aren't covered?
3. **Notice waste** - What guidance didn't help?

### Update skill mid-flight
Don't wait until done. When you hit friction:
- Add guidance that would have helped
- Add a failure mode entry
- Add or refine an eval check

### Draft eval checks
Eval checks should be runnable where possible:
- `grep` for pattern violations
- `wc -l` for length constraints
- `jq` for schema validation
- Checklists for subjective criteria

### Complete Target 1
Keep iterating until output is good. "Good" means:
- You'd ship it
- It demonstrates skill's intent
- You learned from making it

### Graduate Exemplar 1
Output 1 is now Exemplar 1. Add to the guide's Exemplars section:
```markdown
## Exemplars

Study before starting:
- [Target 1 location/link]
```

---

## Phase 3: Spin the Flywheel

### Pick Target 2
Different enough to stress-test the skill:
- Different scale?
- Different constraints?
- Different edge cases?

### Co-develop again
- Skill v1 + Target 2
- Notice what v1 handles well (validate)
- Notice what v1 misses (extend)
- Update skill as you go

### Graduate Exemplar 2
Now you have two reference points. Patterns become clearer.

### Continue until mature
Keep spinning. Each turn:
- Skill changes shrink (converging)
- New targets reveal fewer gaps
- Outputs pass eval earlier
- Others can use the skill successfully

---

## Structure Emergence

Don't design structure upfront. Let it emerge from friction.

| Signal | Response |
|--------|----------|
| Keep hitting same failure | Add eval check |
| Output quality varies | Add template or constraint |
| Guidance feels like overhead | Remove or loosen it |
| Decisions feel arbitrary | Add rubric criteria |
| Others misuse the skill | Add "Things to get right" |

The right level of structure is discovered, not designed.

---

## Eval Loop Mechanics

Every skill should encode its own eval loop.

### Eval check anatomy
```markdown
| # | Check | Pass criteria | If fail |
|---|-------|---------------|---------|
| 1 | [Name] | [Runnable test or clear criterion] | [Specific fix] |
```

### Runnable checks (prefer these)
```bash
# Pattern violation
grep -n "bad pattern" file && echo "FAIL" || echo "PASS"

# Line count
wc -l file  # compare to target range

# Schema validation
jq '.required_field' file.json > /dev/null && echo "PASS"
```

### Subjective checks (when necessary)
- Clear rubric criteria
- Binary pass/fail, not scores
- Specific fix action if fail

### The loop
```
while not all_checks_pass:
    run_checks()
    for each failure:
        apply_fix()
    re-run_checks()
```

---

## Failure Modes Table

Catalog failures as you discover them. This is gold.

```markdown
## Failure Modes

| Symptom | Cause | Fix |
|---------|-------|-----|
| [What you observe] | [Why it happened] | [How to fix] |
```

Grow this table with each flywheel turn. It becomes:
- Debugging guide for users
- Training data for the skill itself
- History of what you learned

---

## Maturity Signals

How to know when the skill is ready:

| Signal | Meaning |
|--------|---------|
| Skill changes shrink | Converging on stable form |
| New targets → no new failure modes | Coverage is good |
| Outputs pass eval on first/second try | Guidance is effective |
| Others succeed with the skill | Not author-dependent |
| Exemplars span the problem space | Skill generalizes |

Mature ≠ frozen. Skills can evolve. But mature means ready for others to use.

---

## Skill Anatomy Reference

```
skills/
  skill-name/
    SKILL.md              # Entry point with frontmatter
    references/
      guide.md            # Detailed process
      [other context]     # Templates, examples, etc.
```

### SKILL.md frontmatter
```yaml
---
name: skill-name           # Identifier
description: One line      # What it does
triggers:                  # Activation phrases
  - trigger phrase
---
```

### Guide structure (adapt as needed)
- Process overview
- Phases with gates
- Templates
- Eval checks table
- Failure modes table
- Exemplar references

---

## Exemplars

Study these before starting a new skill:

- **llms-txt** (`skills/llms-txt/`) - Demonstrates eval checks, failure modes, templates.

**In progress:**
- **ruthless-reviewer** (`skills/ruthless-reviewer/`) - Target 1, being co-developed with this skill.

---

## Recursive Validation

This skill should improve itself.

The skill-crafting skill was bootstrapped by:
- Reflecting on what made llms-txt effective (informing exemplar)
- Drafting v0 based on those learnings
- Using ruthless-reviewer to identify gaps
- Iterating until the skill meets its own bar

Current status:
- v0: Initial draft
- Target 1: ruthless-reviewer skill (in progress)
- [Target 2 pending]

If this skill can't improve itself, it doesn't work.

---

## Self-Eval Checks

This skill must pass its own criteria.

| # | Check | Command | Pass |
|---|-------|---------|------|
| 1 | Has SKILL.md with frontmatter | `head -1 SKILL.md \| grep -q "^---"` | Exit 0 |
| 2 | Has guide.md | `test -f references/guide.md` | Exit 0 |
| 3 | Guide has failure modes table | `grep -q "## Failure Modes" references/guide.md` | Exit 0 |
| 4 | Guide has eval checks | `grep -q "## Self-Eval Checks\|## Eval Checks" references/guide.md` | Exit 0 |
| 5 | Has exemplar reference | `grep -q "## Exemplar" references/guide.md` | Exit 0 |

```bash
# Run from skill directory
cd skills/skill-crafting
echo "1) SKILL.md frontmatter:" && head -1 SKILL.md | grep -q "^---" && echo "PASS" || echo "FAIL"
echo "2) guide.md exists:" && test -f references/guide.md && echo "PASS" || echo "FAIL"
echo "3) Failure modes:" && grep -q "## Failure Modes" references/guide.md && echo "PASS" || echo "FAIL"
echo "4) Eval checks:" && grep -qE "## (Self-Eval|Eval) Checks" references/guide.md && echo "PASS" || echo "FAIL"
echo "5) Exemplar ref:" && grep -q "## Exemplar" references/guide.md && echo "PASS" || echo "FAIL"
```

**Current status:**
| # | Status |
|---|--------|
| 1 | Pass |
| 2 | Pass |
| 3 | Pass |
| 4 | Pass |
| 5 | Pass (llms-txt referenced) |

## Failure Modes

| Symptom | Cause | Fix |
|---------|-------|-----|
| Skill has no eval loop | Skipped eval checks in Phase 2 | Add checks before graduating |
| Output quality varies | No templates or constraints | Add structure where variance hurts |
| Others can't use the skill | Author-dependent tacit knowledge | Add exemplars and failure modes |
| Skill changes every target | Not converging | Check if problem is well-scoped |
| Over-specified structure | Designed upfront, not emerged | Remove what doesn't prevent failures |

---

## Quick Start

1. Create `skills/NAME/SKILL.md` with frontmatter
2. Create `skills/NAME/references/guide.md` with rough process
3. Pick a real target
4. Co-develop: update skill as you work the target
5. Graduate output to exemplar
6. Spin flywheel with Target 2
7. Continue until mature
