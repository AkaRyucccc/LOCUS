# LOCUS Agent Rules — V2

You are an engineering agent working on LOCUS.

These rules govern how you interpret project documentation, change code, handle data, protect user information, validate work, and report completion.

---

## 1. Product Authority and Terminology

### 1.1 Product end user

The LOCUS product end-user persona is **The Observer**.

"The Observer" refers only to the person using the LOCUS product experience. It does not refer to the developer, repository owner, product manager, or the person issuing instructions to the engineering agent.

Within immersive scenes, the product end user observes and explores cultural memory rather than replacing a historical, literary, or artistic character.

### 1.2 Product principle

LOCUS transforms physical locations into portals to cultural memory.

Directional product principle:

**Museum × Cinema × GIS × Exploration**

This statement is a product north star, not an implementation specification.

Concrete UI, interaction, motion, layout, and visual decisions must follow `docs/DESIGN_SYSTEM.md` and `docs/UX_FLOW.md`.

LOCUS must not drift into a generic:

- Google Maps clone
- TripAdvisor clone
- Wikipedia reader
- SaaS dashboard
- AI chatbot
- conventional tourism directory

### 1.3 Core loop and module ownership

The product loop is:

Location
→ Cultural Match
→ Enter Scene
→ Walk
→ Discover

Current module mapping:

- **Location** → `components/globe`, `components/search`, `services/geo`
- **Cultural Match** → `services/culture`, `services/matching`, `components/match`
- **Enter Scene** → `services/scene`, `components/scene`
- **Walk** → `components/map`, future routing services
- **Discover** → future recommendation / exploration services

Do not implement modules belonging to a future loop stage unless that stage is explicitly in the Current Sprint.

---

## 2. Documentation Loading and Priority

### 2.1 Always read before modifying code

Always read:

1. `docs/AGENT_RULES.md`
2. `docs/SPRINT.md`

The **Current Sprint** is the section explicitly marked `Current Sprint` in `docs/SPRINT.md`.

### 2.2 Read task-relevant documentation

For UI, interaction, motion, or presentation work, also read:

- `docs/PRODUCT.md`
- `docs/UX_FLOW.md`
- `docs/DESIGN_SYSTEM.md`

For data, matching, geographic, or persistence work, also read:

- `docs/DATA_SCHEMA.md`
- `docs/ARCHITECTURE.md`

For architecture, framework, dependency, API, or service-boundary work, also read:

- `docs/ARCHITECTURE.md`
- `docs/DATA_SCHEMA.md` when data contracts are affected

### 2.3 Documentation priority

When instructions appear to conflict, use this order:

1. Explicit approved task instruction for the current task
2. `docs/AGENT_RULES.md`
3. `docs/SPRINT.md`
4. `docs/PRODUCT.md`
5. `docs/DATA_SCHEMA.md` and `docs/ARCHITECTURE.md`
6. `docs/DESIGN_SYSTEM.md`
7. `docs/UX_FLOW.md`
8. Existing implementation

An explicit task instruction does **not** override security, privacy, licensing, provenance, or destructive-operation restrictions in this document.

Do not silently resolve material contradictions between project documents.

If a material conflict exists, stop that part of the work and report:

- conflicting files / instructions
- exact conflict
- likely impact
- recommended resolution

Do not choose a new product or architectural direction on your own.

---

## 3. Sprint Scope and Change Control

### 3.1 Current Sprint is authoritative

Only implement work that belongs to the Current Sprint or is explicitly approved as an exception.

If a requested change falls outside the Current Sprint:

- do not implement it automatically
- identify why it is out of scope
- identify which sprint or module it belongs to
- wait for explicit approval before expanding scope

### 3.2 Reversible vs material decisions

You may make small, reversible implementation decisions without approval, such as:

- local variable names
- private helper functions
- small component extraction
- internal file organization that does not alter documented architecture
- test organization

You must request approval before making a material decision affecting:

- framework or rendering engine
- database or persistence model
- public API or shared interface contract
- core data schema
- privacy or retention behavior
- licensing or content-rights behavior
- paid API provider or material recurring cost
- major dependency
- core UX flow
- sprint scope
- repository visibility or settings

### 3.3 Architecture changes

Do not replace an established architectural pattern, framework, rendering approach, storage model, or major dependency without first proposing the change and receiving explicit approval.

A proposal must include:

- reason for change
- expected benefit
- risks
- migration cost
- affected files / modules
- alternatives considered
- rollback path when relevant

### 3.4 Dependencies

A dependency already explicitly required by `ARCHITECTURE.md` or the Current Sprint may be installed.

Before adding any other runtime dependency, report:

- package name and version
- purpose
- whether an existing dependency can solve the problem
- bundle / runtime impact
- license
- maintenance status
- alternatives considered

Wait for approval before installing it.

Development-only dependencies required to satisfy the Current Sprint's documented lint, test, or build tooling may be added when clearly necessary, but they must still be reported in the completion summary.

---

## 4. Data Integrity and Provenance

### 4.1 Never invent source facts

Never fabricate:

- cultural records
- creator identities
- dates
- geographic coordinates
- source provenance
- rights status
- quotations
- historical relationships
- confidence values presented as verified measurements

If a factual value is unavailable:

- use `null`, `unknown`, or another schema-approved missing state
- preserve uncertainty explicitly
- do not ask an LLM to fill the gap
- do not convert inference into a verified fact

### 4.2 Source tracking

Verified cultural data must be traceable to one or more `SourceReference` records.

Every source-backed record should preserve, where available:

- `sourceId`
- `sourceType`
- `sourceUrl`
- `retrievedAt`
- `confidence`
- relevant external identifier

Generated content must preserve which source records it was based on.

### 4.3 Synthetic / fixture data

Synthetic data is allowed only for:

- tests
- fixtures
- demos
- explicitly marked local mock datasets

Synthetic records must contain:

`isSynthetic: true`

Synthetic records must never be silently mixed with verified production cultural data.

Test or demo interfaces should make synthetic status inspectable.

### 4.4 Generated content

Generated scene or narrative data must contain:

`generated: true`

Generated output must never overwrite verified source fields.

Generated records should include provenance linking them back to the verified source IDs used as inputs.

The product must be able to distinguish:

- verified source facts
- transformed / generated interpretation

### 4.5 Coordinate rules

LLMs must never be used as an authoritative source of coordinates.

Coordinates may come only from an approved source such as:

- device GPS
- a geocoding provider
- a verified external dataset
- a manually verified project entry

Coordinate records must preserve:

- coordinate source
- source identifier when available
- precision / accuracy when known
- whether the coordinate is exact or approximate

### 4.6 Deterministic geographic calculations

Use documented deterministic geographic methods for distance calculations, such as Haversine or a reviewed geospatial library.

Ranking must be deterministic for identical inputs.

When scores tie, use a documented stable tie-break order. Initial default:

1. final score
2. geographic confidence
3. cultural significance
4. shorter distance
5. stable object ID

Any randomness used in tests, demos, or ranking experiments must use an explicit reproducible seed.

---

## 5. Privacy, Secrets, and Sensitive Location Data

### 5.1 Secrets

Never commit or expose:

- API keys
- access tokens
- passwords
- private certificates
- service credentials
- secret environment values

Local secret files such as `.env.local` must remain untracked.

Provide `.env.example` only with placeholder values when configuration documentation is needed.

### 5.2 Precise user location

Precise end-user latitude / longitude is sensitive operational data.

Default MVP behavior:

- keep precise location client-side when server processing is not required
- do not persist precise user location by default
- do not add analytics that capture precise location without explicit product approval

Before introducing server transmission or persistence of precise location, document and obtain approval for:

- purpose
- fields transmitted
- retention period
- storage system
- deletion behavior
- third parties receiving the data

Do not log precise user coordinates in production logs unless explicitly approved and justified.

---

## 6. Copyright, Licensing, and Cultural Media

Do not assume an artwork, image, text, audio file, video, map asset, or archival object is reusable merely because it is old or publicly visible online.

Cultural media records must preserve rights metadata defined in `DATA_SCHEMA.md`, including:

- rights status
- license when known
- source
- attribution requirement
- rights verification state

If reuse rights are unknown, treat the asset as unavailable for production use until verified.

Do not strip required attribution.

Do not replace unknown rights information with an inferred license.

Generated media must remain distinguishable from source media in metadata.

---

## 7. Engineering Discipline

### 7.1 Business logic

Keep deterministic business logic outside presentation components.

UI components should not become the authoritative implementation of:

- geographic distance
- ranking
- provenance rules
- rights rules
- data validation
- persistence policy

### 7.2 Type safety

Do not weaken TypeScript strictness to make code pass.

Do not introduce broad `any` usage to bypass type errors.

Prefer explicit domain types aligned with `DATA_SCHEMA.md`.

### 7.3 Error handling

Do not silently substitute fabricated values when external data or services fail.

Use explicit:

- error states
- unavailable states
- empty states
- retry paths where appropriate

### 7.4 Accessibility baseline

At minimum:

- interactive controls require accessible names
- keyboard interaction must not be intentionally broken
- motion must respect `prefers-reduced-motion`
- essential information must not depend only on animation
- provide a usable fallback when WebGL is unavailable

### 7.5 Mobile performance baseline

LOCUS is mobile-first.

Avoid:

- unnecessary re-renders
- loading full-resolution media before needed
- excessive simultaneous globe labels / objects
- blocking main-thread work during interaction

Primary target viewport remains documented in `DESIGN_SYSTEM.md`.

Performance optimization must not silently degrade core data integrity or accessibility.

---

## 8. Git and Repository Safety

Allowed unless the task says otherwise:

- create and edit project files
- add tests
- run local validation commands
- create normal commits when the execution environment requires commits

Do not perform without explicit approval:

- force push
- rewrite Git history
- `git reset --hard`
- destructive mass deletion
- delete branches
- change repository visibility
- change repository settings
- alter branch protection
- remove large parts of working code unrelated to the task

Use clear scoped commit messages when commits are created, for example:

- `feat(globe): add opening globe shell`
- `fix(match): stabilize cultural ranking`
- `docs(agent): clarify provenance rules`

---

## 9. Validation and Anti-Gaming Rules

### 9.1 Commands

Use the package manager declared by the repository.

Once Sprint 00 establishes canonical scripts, use the repository-defined equivalents of:

- lint
- test
- build

Do not switch package managers without approval.

### 9.2 Do not game validation

Never make validation pass by:

- deleting existing tests
- skipping or disabling existing tests
- weakening assertions solely to hide a defect
- disabling lint rules solely to hide errors
- disabling strict TypeScript
- globally suppressing errors
- removing required build steps
- modifying validation configuration solely to conceal failures

Changing test or lint configuration is allowed only when the Current Sprint explicitly requires configuring that tooling or when there is a documented defect in the configuration. Explain the change.

### 9.3 Existing failures

If validation already fails before your changes:

- identify the pre-existing failure
- distinguish it from failures introduced by your work
- do not claim full validation success
- do not silently repair unrelated failures unless approved or required for the Current Sprint

### 9.4 Repeated failure

If the same failure remains after three materially different fix attempts:

stop that repair loop and report:

- failing command
- relevant error
- root-cause hypothesis
- three approaches attempted
- remaining uncertainty
- recommended next action

Do not retry indefinitely.

---

## 10. Acceptance Criteria

The authoritative acceptance criteria are the bullets under the Current Sprint's **Acceptance Criteria** section in `docs/SPRINT.md`, plus any explicit acceptance criteria in the approved task instruction.

Before declaring completion:

- evaluate every acceptance criterion individually
- mark each one pass / fail / blocked
- provide evidence where possible

Do not claim the sprint or task is complete while required acceptance criteria remain failed or blocked.

---

## 11. Documentation Synchronization

Documentation updates are part of the same task when implementation changes documented behavior.

If implementation changes:

- data schema → update `docs/DATA_SCHEMA.md`
- architecture or service boundaries → update `docs/ARCHITECTURE.md`
- UX behavior → update `docs/UX_FLOW.md`
- visual tokens or interaction rules → update `docs/DESIGN_SYSTEM.md`
- sprint scope or acceptance criteria → update `docs/SPRINT.md`
- agent governance → update `docs/AGENT_RULES.md`

Do not allow code and authoritative documentation to knowingly diverge.

---

## 12. Completion Protocol

Before declaring a task complete:

1. confirm the Current Sprint and task scope
2. run the repository-defined lint command
3. run the repository-defined test command
4. run the repository-defined production build command
5. evaluate each acceptance criterion
6. confirm no prohibited validation shortcuts were used
7. update relevant documentation
8. update `CHANGELOG.md` only after implementation and validation status are known

Use a Keep-a-Changelog-style entry under `Unreleased` where practical.

Do not automatically begin the next sprint.

### Required completion report

Return a concise structured report containing:

1. **Scope completed**
2. **Files changed**
3. **Dependencies added / removed**, including reason
4. **Commands executed**
5. **Validation results**
6. **Acceptance criteria checklist** — pass / fail / blocked
7. **Documentation updated**
8. **Assumptions**
9. **Known risks / unresolved items**
10. **Recommended next task**

Never report success for a command that was not actually executed.

Never report a test, lint, build, data source, license, or verification result that was not actually observed.
