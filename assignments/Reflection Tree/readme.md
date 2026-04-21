# Daily Reflection Tree — DT Fellowship Submission

A deterministic end-of-day reflection tool. No LLM at runtime. Same answers → same path → same reflection. Every time.

---

## Repository Structure

```
/tree/
  reflection-tree.json     ← Part A: complete tree data (37 nodes)
  tree-diagram.md          ← Part A: visual diagram (Mermaid)

/transcripts/
  persona-1-transcript.md  ← Part B: Victor / Contributing / Altrocentric
  persona-2-transcript.md  ← Part B: Victim / Entitled / Selfcentric

write-up.md                ← Part A: design rationale (2 pages)
README.md                  ← this file
```


## How to Read the Tree

`reflection-tree.json` is a flat array of node objects. Each node has:

| Field | Purpose |
|-------|---------|
| `id` | Unique node identifier |
| `type` | Node behaviour: `start`, `question`, `decision`, `decision_signal`, `reflection`, `bridge`, `summary`, `end` |
| `text` | What the user sees. `{NODE_ID.label}` is replaced with the user's earlier answer at runtime. |
| `options` | Fixed choices (question nodes only). Each option can carry a `signal`. |
| `target` | Next node ID (for non-branching nodes) |
| `conditions` | Routing rules (for decision nodes). See below. |
| `signal` | Tally tag written to state: e.g. `axis1:internal` increments `state.signals.axis1.internal` |
| `axis` | Which reflection axis this node belongs to (`axis1`, `axis2`, `axis3`) |

### Decision Node Types

**`decision`** — routes based on the *literal value* of the previous question's answer:
```json
{ "condition": "sunny|unpredictable", "target": "A1_Q2_HIGH" }
```
Read: "If the previous answer value was 'sunny' OR 'unpredictable', go to A1_Q2_HIGH."

**`decision_signal`** — routes based on which signal pole is *dominant* (has more tallies) for the given axis:
```json
{ "condition": "internal", "target": "A1_Q3_INTERNAL" }
```
Read: "If axis1.internal > axis1.external, go to A1_Q3_INTERNAL."

A tie resolves to "mixed" — currently routes to the first condition.

---

## Node Inventory

| Type | Count | Meets requirement? |
|------|-------|-------------------|
| `start` | 1 | ✓ |
| `question` | 15 | ✓ (req: 8+, at least 2 per axis) |
| `decision` / `decision_signal` | 9 | ✓ (req: 4+) |
| `reflection` | 6 | ✓ (req: 4+, at least 1 per axis) |
| `bridge` | 2 | ✓ (req: 2+) |
| `summary` | 1 | ✓ |
| `end` | 1 | ✓ |
| **Total** | **35** | ✓ (req: 25+) |

---

## Psychological Sources

| Axis | Concept | Source |
|------|---------|--------|
| Axis 1 | Locus of Control | Rotter, J.B. (1954). *Social Learning and Clinical Psychology* |
| Axis 1 | Growth Mindset | Dweck, C.S. (2006). *Mindset: The New Psychology of Success* |
| Axis 2 | Psychological Entitlement | Campbell, W.K. et al. (2004). *Journal of Personality* |
| Axis 2 | Organisational Citizenship Behaviour | Organ, D.W. (1988). *Organizational Citizenship Behavior* |
| Axis 3 | Self-Transcendence | Maslow, A.H. (1969). *Journal of Transpersonal Psychology* |
| Axis 3 | Perspective-Taking | Batson, C.D. (2011). *Altruism in Humans* |

---

## The Three Axes — Quick Reference

**Axis 1: Locus (Victim ↔ Victor)**
- Internal pole: person sees their own choices and adaptations as shaping outcomes
- External pole: person sees circumstances, others, or luck as the primary drivers
- Questions surface: how they narrate what went well or badly, whether they can identify a choice they made

**Axis 2: Orientation (Entitlement ↔ Contribution)**
- Contribution pole: attention flows toward what they gave, helped, or enabled
- Entitlement pole: attention flows toward recognition received, credit, fairness of exchange
- Questions surface: the direction of attention during interactions, whether discretionary effort was given

**Axis 3: Radius (Self-Centric ↔ Altrocentric)**
- Altrocentric pole: the employee's mental model of the day includes others — their experience, state, impact
- Self-centric pole: the day is narrated primarily as a personal experience
- Questions surface: whose experience they're replaying, whether they sought to understand others' perspective

---

## Design Constraints Respected

- ✅ No LLM calls at runtime — pure tree traversal
- ✅ Deterministic: same answers always produce the same path
- ✅ No free-text input — all questions have fixed options only
- ✅ Reflections use interpolation (`{NODE_ID.label}`) not generated text
- ✅ No moralizing — reflections observe and nudge, they don't grade
- ✅ Axes flow in sequence (1 → 2 → 3) with connecting bridges
- ✅ Tree is readable as data — any developer can trace paths from the JSON without running code
