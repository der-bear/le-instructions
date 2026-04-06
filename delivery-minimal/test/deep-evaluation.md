# Delivery Minimal — 10-Round Deep Evaluation

This file records the static audit used to evolve `delivery-minimal/` after comparing it against both:
- `delivery-original-stabilized/`
- `delivery-rework/`

The goal was not to copy either version wholesale. The goal was to keep the high-value reliability nuances and reject the parts that created avoidable stalls, skipped tool calls, or brittle UI dependencies.

## Round 1 — Outer Architecture
Decision:
- Keep the same platform-facing architecture:
  - `action/`
  - `system/`
  - `resources/`
  - `test/`

Adopted from previous packs:
- stable file/folder conventions
- 3 action entrypoints for compatibility

Rejected:
- introducing a completely new packaging model

## Round 2 — Global Instruction Scope
Decision:
- Keep `system/min-1-global.md` as general policy only.
- Move phase mechanics into phase files.

Adopted:
- stabilized-style hierarchy and persistence
- general summarization and resource-loading policy
- DTO/native-object discipline

Rejected:
- phase-specific branching logic in the global file
- verbose prompt scripting in the global file

## Round 3 — Summarization And Context Passing
Decision:
- Restore `ANCHOR + summarize_history + next_instructions` for major scalar phase handoffs.
- Do not reintroduce summary-review phases.

Adopted:
- stabilized summary tags and anchor pattern
- automatic context transfer across phases

Rejected:
- mid-flow review cards
- "Continue" summary gates

Exception kept:
- no summarization in the method router before webhook
- no summarization in the criteria builder before returning to account creation

Reason:
- both exceptions need native structured values to stay in working memory.

## Round 4 — Phase 0 Selection Flows
Decision:
- Carry over stabilized/rework empty-list handling, typed fallback matching, and profile loading.

Adopted:
- `get_clients`
- `get_client`
- `get_delivery_methods`
- empty-list stop conditions
- ambiguous typed-input retry
- summary handoff after successful selection

Rejected:
- summary-free transitions between these phases

## Round 5 — Phase 1 Client Creation
Decision:
- Keep the stabilized client-create payload and defaults.
- Reintroduce summary handoff to Phase 2.

Adopted:
- `clientStatus="New"`
- `clientAutomationType="Price"`
- username defaults to email
- PST default timezone for creation

Rejected:
- carrying the generated password as durable visible state between phases

## Round 6 — Phase 2 Lead Type Selection
Decision:
- Prefer `display_lead_types_choice` for compatibility.
- Keep typed fallback only as backup.
- Summarize after selection.

Adopted:
- stabilized tool-first lead-type selector
- rework typed fallback idea

Rejected:
- manual card-building as the primary selector path

## Round 7 — Phase 3 Delivery Method Flow
Decision:
- Keep a dedicated Phase 3 router.
- Split Portal, Email, FTP, and Webhook into method-specific resources.
- Let each method-specific resource collect and build the delivery schedule locally.
- Reintroduce summary after successful method creation for full setup.

Adopted:
- rework router split
- method-specific create resources for Portal, Email, and FTP
- self-contained method branches that each own schedule parsing and method payload construction
- stabilized deliveryDays parsing rules
- stabilized create payloads for Portal/Email/FTP/Webhook

Rejected:
- connection-test phases
- method-summary review phase

Exception kept:
- no summarization in the router before loading a method-specific resource

Reason:
- self-contained method branches are more stable than native `deliveryDays` handoff across resources.

## Round 8 — Webhook Nuance
Decision:
- Keep the rework's bounded webhook branch shape, but strip table previews and connection testing.

Adopted:
- URL normalization
- `Provide instructions` vs `Skip mapping`
- size/URL-only input rejection
- one auto-detect confirmation
- blocking clarification for ambiguous mappings
- summary handoff after creation for full setup

Rejected:
- mapping preview tables
- connection-test handoff
- summary-review phase after method creation

## Round 9 — Phase 4 Account Creation
Decision:
- Keep the fixed visible input order.
- Restore summary handoff after account creation for full setup.
- Keep single-create account behavior.

Adopted:
- stabilized single `create_delivery_account` mutation
- rework/review finding that geography must be collected before hidden discovery
- explicit `All states` vs `Specific states`

Rejected:
- create-then-update criteria path as the normal flow
- mandatory state targeting for every buyer
- account summary review phase

## Round 10 — Criteria Builder And Activation
Decision:
- Keep a small optional criteria builder.
- Keep activation as the terminal phase with full `update_client` DTO.

Adopted:
- rework nuance that `update_client` is a replace-style mutation and must resend all fields
- optional, pre-create criteria gathering
- terminal activation choice with retry

Rejected:
- mutating the account inside the criteria builder
- summarizing the criteria builder before returning to Phase 4

Reason:
- `compiledCriteria` must stay native for the immediate `create_delivery_account` call.

## Final Evaluation Result
The resulting `delivery-minimal/` pack intentionally combines:
- stabilized-style summary-based scalar context transfer
- rework-style flow decomposition where it improved reliability
- explicit exceptions for the two structured-object handoffs where summary compression would likely hurt fidelity

Net position:
- same outer architecture
- thinner semantics
- restored context passing
- fewer brittle UI dependencies
- no separate summary/review phases
