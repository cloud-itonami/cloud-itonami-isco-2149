# cloud-itonami-isco-2149

**ISCO-08 2149: Engineering Professionals Not Elsewhere Classified**

A langgraph-clj StateGraph actor for general engineering support. This actor drafts and prepares engineering analysis material for licensed engineers' review and professional sign-off in engineering specialties not covered by the fleet's more specific engineering actors.

## Architecture

The actor is a closed-loop state machine with four roles:

- **Advisor** (`geneng.advisor`) — proposes engineering operations (draft analysis, log project data, flag safety risks, request client review)
- **Governor** (`geneng.governor`) — applies independent safety/compliance checks (project registration, no direct actuation, no certification issuance)
- **Store** (`geneng.store`) — append-only audit ledger + project registry
- **StateGraph** (`geneng.actor`) — orchestrates intake → advise → govern → decide → commit/hold/escalate

## Domain constraints

This is a **support role**, not a sign-off authority:

- It **drafts** engineering analysis and calculations for human review
- It **never** issues certified engineering designs
- It **never** issues safety certifications (licensed engineer exclusive)
- It **always** escalates safety risks to human review
- It **always** holds on low confidence (< 0.6)

## Operations

All `:effect :propose` (never direct writes):

- `:draft-engineering-analysis` — engineering analysis/calculation draft
- `:log-project-data` — project data logging
- `:flag-safety-risk` — surface a safety/technical risk (always escalates)
- `:request-client-review` — propose client review scheduling

## Testing

```bash
kbb -M:test
```

## License

AGPL-3.0-or-later
