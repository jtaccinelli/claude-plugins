# Request: <request-slug>

**Date**: <YYYY-MM-DD>
**Status**: draft | approved

---

## Request

<the request in plain language, as given by the user>

---

## Entry Nodes

- `<concern> · <layer>` — <why this is an entry point>

---

## Contract Graph

| Contract | Consumer | Provider | Verdict | Status |
|---|---|---|---|---|
| `<name>` | `<node · item>` | `<node>` | Met / Partial · additive / Partial · breaking / Unmet / Countered | satisfied / create / escalated |

---

## Met / Modify / Create Breakdown

**Met** (no build needed):
- `<node · item>`

**Modify** (Partial · additive / Partial · breaking):
- `<node · item>` — <what changes, and which verdict>

**Create** (Unmet, passed the reuse guardrails):
- `<node · item>` — <why it passed: significant+consistent / 2+ consumers this pass / 3+ historical requests>

---

## Build Waves

*(A first pass — refined for real by the coordinator's Sequence Build Waves capability once this plan is approved.)*

1. **Wave 1** (Globals + Elements): `<items>`
2. **Wave 2**: `<items>`

---

## Escalations

<any unresolved consumer/provider conflicts surfaced to the user via AskUserQuestion, and how each was resolved. Omit this section entirely if none occurred.>
