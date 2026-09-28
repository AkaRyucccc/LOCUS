# LOCUS Agent Rules

You are an engineering agent working on LOCUS.

Before modifying code, always read:

1. docs/PRODUCT.md
2. docs/UX_FLOW.md
3. docs/DESIGN_SYSTEM.md
4. docs/DATA_SCHEMA.md
5. docs/ARCHITECTURE.md
6. docs/SPRINT.md

## Product Principle

LOCUS transforms physical locations into portals to cultural memory.

The product must feel like:

Museum × Cinema × GIS × Exploration.

It must NOT feel like:

Google Maps,
TripAdvisor,
Wikipedia,
a SaaS dashboard,
or a generic AI chatbot.

## Core Loop

Location
→ Cultural Match
→ Enter Scene
→ Walk
→ Discover

## User Role

The user is always **The Observer**.

## Engineering Rules

- Do not implement features outside the active sprint.
- Do not redesign unrelated components.
- Do not introduce a new dependency unless necessary.
- Do not replace working architecture without explaining why.
- Never fabricate cultural records.
- Never fabricate geographic coordinates.
- Never treat LLM output as verified cultural evidence.
- Keep cultural source data separate from generated scene data.
- Keep business logic outside UI components where practical.
- Prefer deterministic logic for geographic calculations and ranking.

## Completion Rules

After implementation:

1. run lint
2. run tests
3. run build
4. verify acceptance criteria
5. update CHANGELOG.md

If acceptance criteria are not met, continue fixing before declaring completion.

Do not automatically begin the next sprint.
