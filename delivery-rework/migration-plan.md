# Delivery-Rework Migration Plan

## Summary

- Rewrite `delivery-rework/` end-to-end into the new phase-local state-machine architecture.
- Preserve current user-facing behavior, prompt wording, branch semantics, and end-state flow unless a change is required to fix a documented correctness bug.
- Keep `delivery/` untouched as the control copy for parity review and measurement.
- Keep the migration low-risk in user-facing behavior while aggressively reducing always-on prompt weight.
- Require additional-agent cross-evaluation at every major stage before proceeding to the next stage.

## Core Architecture Rules

### 1. System prompt minimization

- Keep global rules only for:
  - instruction precedence
  - phase-local authority
  - ASK/SUGGEST semantics
  - tool-discipline invariants
  - prompt exactness
  - technical transparency
  - off-topic handling
  - quality gate
  - summary-format parsing
  - resource-loading / next-resource execution
- Move phase-specific behavior out of global files:
  - delivery-type-specific normalization
  - schedule parsing
  - state parsing
  - criteria parsing
  - local retry logic
  - local selector rendering rules
  - local repair branches
  - local DTO cautions if tool-schema help is not yet available

### 2. Tool-schema ownership

- If a tool can expose helper instructions through its schema or tool metadata, move tool-specific usage guidance there instead of keeping it in the global prompt.
- Primary candidates:
  - `display_adaptive_card` rendering conventions
  - `summarize_history` payload format expectations
  - DTO-shape rules for create/update tools
  - distinction between data tools and resource-loading tools
- Until external tool-schema support exists, keep only the minimum required tool detail in the owning phase file, not in global files.

### 3. Phase authoring format

- Every rewritten action/phase file uses one explicit state-machine format:
  - objective
  - phase-local rules
  - explicit state evaluation
  - explicit state transitions
  - explicit `STOP AND YIELD` points
  - explicit summary / handoff requirements
- For mutually exclusive branches, use `IF / ELSE IF / ELSE` wording instead of parallel `IF` blocks wherever it improves clarity.
- Use one consistent card verb:
  - `DISPLAY [adaptive_card]` for rendered cards
  - `ASK [conversational]` only for plain-text user input

### 3a. Conditional resource decomposition

- Prefer conditional resource loading when one branch family is materially more complex than its siblings.
- Keep simple shared setup in a short router resource, then hand off to branch-specific resources only when that reduces branch density enough to improve traceability.
- Do not default to one-resource-per-branch if the branch is short and mechanically obvious. Extra resource hops have measurable cost.
- Default ranking for branch-heavy phases:
  1. router plus complex-branch split
  2. one-resource-per-method only when multiple methods are independently complex
  3. monolithic phase with explicit flags only when the branch graph remains easy to validate end to end
- If a phase stays monolithic, every repair loop must use named retained state flags. Do not rely on vague conditions like "the user has not yet answered" without a traceable state marker.

### 4. Summary / handoff contract

- Support both during migration:
  - legacy `<summary><retain><next_phase>`
  - structured:
    - `# Current System State`
    - `# Next Instructions`
    - `NEXT RESOURCE: mcp://resource/...`
- All rewritten phases should author the structured format.
- Every rewritten summary must be self-contained. No rewritten phase may rely on implicit retained-state carry-forward.

### 5. Shared versus branch-local logic

- Keep shared across sibling branches only when the behavior is truly identical:
  - summary structure
  - handoff mechanics
  - common card-template shell shape
  - common create-tool failure wording when the retry behavior is the same
- Keep branch-local when the behavior changes by method, entity type, or repair loop:
  - input normalization
  - DTO composition
  - eligibility rules
  - validation / parse repair
  - mapping logic
  - criteria parsing
  - confirmation and retry branches
- Avoid pseudo-shared abstractions that hide branch differences. Shared logic should reduce repetition without making the branch graph harder to trace.

## Standing Quality Gates

These checks apply at every stage, not just at the end.

### Workflow integrity gate

- Trace the full reachable branch graph for the stage being rewritten.
- Write or update a compact branch matrix for the stage before locking it.
- Confirm:
  - entry conditions
  - re-entry behavior
  - invalid-input repair
  - partial-state repair
  - empty-result behavior
  - terminal behavior
  - summary ownership
  - next-phase handoff correctness
- No phase may depend on hidden context bleed from a previous phase once rewritten.

### Optimality gate

- Measure the rewritten stage against:
  1. the original `delivery/` version
  2. the current `delivery-rework/` pre-stage baseline
  3. the shared example patterns discussed in chat
- Evaluate:
  - prompt weight reduction
  - global-vs-local instruction placement
  - branch clarity
  - retry-loop containment
  - number of assistant turns on the happy path
  - likely card-generation / summary overhead
- Do not accept a stage if it is cleaner stylistically but weaker in branch coverage or parity.

### Cross-agent review gate

- Before locking each major stage, use additional agents for:
  - workflow-integrity trace
  - prompt-weight / optimization review
  - authoring-consistency review
- When a stage introduces conditional resource splits, include at least one review that challenges whether the extra resource hop is worth the added branch clarity.
- Capture findings and fold them into the next patch before moving on.

## Implementation Stages

### Stage 1: System layer

Files:
- `delivery-rework/system/1-global.md`
- `delivery-rework/system/2-flow-specific.md`

Goals:
- shrink the system layer
- keep only cross-cutting rules global
- define `STOP AND YIELD`
- define dual summary parsing
- keep resource-transition mechanics only in `2-flow-specific.md`

Stage checks:
- no delivery-type-specific normalization left in `1-global.md`
- no duplicated resource-transition rules across both system files
- no tool-detail blocks that belong in tool schema/help instead

### Stage 2: Entry and early routing

Files:
- `delivery-rework/action/*.md`
- `delivery-rework/resources/phase-0-select-client.md`
- `delivery-rework/resources/phase-0-select-client-and-method.md`
- `delivery-rework/resources/phase-1-create-client.md`
- `delivery-rework/resources/phase-2-get-lead-types.md`

Goals:
- preserve current routing semantics
- keep add-account bypass of Phase 2
- make selection retries stay inside the phase
- remove summarize/reload retry loops on valid input
- make summaries self-contained

Stage checks:
- Phase 2 valid selection completes in one in-phase render/ack/handoff path
- no rewritten early phase relies on prior hidden retained state
- action routing branches use explicit authored state logic

### Stage 3: Delivery method unit

Files:
- `delivery-rework/resources/phase-3-create-delivery-method.md`
- `delivery-rework/resources/phase-3-webhook-delivery-method.md`
- `delivery-rework/resources/phase-3-simple-delivery-methods.md`
- `delivery-rework/resources/phase-3b-test-connection.md`
- `delivery-rework/resources/phase-4-delivery-method-summary.md`

Goals:
- preserve schedule, delivery-type, mapping, schema-repair, and connection-test UX
- split the delivery-method unit by concern:
  - keep schedule + delivery-type selection in the router
  - keep webhook configuration and repair loops isolated in their own resource
  - keep Portal / Email / FTP together in a simpler sibling resource unless later evidence justifies splitting FTP again
- move schedule parsing and delivery-type-specific normalization into Phase 3
- fix FTP protocol handling
- replace webhook-vs-portal connection-test heuristics with explicit retained capability state
- ensure Phase 3b re-retains everything Phase 4 and Phase 5 need

Stage checks:
- the router owns only shared schedule/type logic; method-specific logic is local to the owning method resource
- each delivery method branch is traceable without reading unrelated branch instructions
- Phase 3 / 3b summary ownership is explicit and lossless
- webhook repair loops are mechanically traceable with explicit state markers or separate resources
- no router resource keeps method-specific repair logic that belongs in the method resource
- field-mapping preview remains visually equivalent
- no global normalization rule is still doing Phase 3’s work

### Stage 4: Delivery account unit

Files:
- `delivery-rework/resources/phase-5-create-delivery-account.md`
- `delivery-rework/resources/phase-5-criteria-builder.md`
- `delivery-rework/resources/phase-6-delivery-account-summary.md`

Goals:
- preserve criteria UX and summary card behavior
- split the delivery-account unit by concern:
  - keep account basics and criteria-context setup in Phase 5
  - let the Phase 5 router own the direct `Skip` branch so that state-only account creation does not pay an extra resource hop
  - move the dense criteria loop and multi-criteria payload creation into a dedicated criteria resource
- remove the embedded duplicate Phase 6 block from Phase 5
- keep only the standalone Phase 6 resource canonical
- fix multi-criteria accumulation
- move state / criteria / enum normalization fully local

Stage checks:
- only one Phase 6 definition exists
- Phase 5 summary carries everything Phases 6-8 need
- multiple criteria survive into the final criteria payload
- the Phase 5 basics resource owns the no-additional-criteria fast path without adding an unnecessary extra resource hop
- the criteria resource owns the additional-criteria loop, payload build, and summary handoff

### Stage 5: Final summary and activation

Files:
- `delivery-rework/resources/phase-7-client-summary.md`
- `delivery-rework/resources/phase-8-activation.md`

Goals:
- preserve both `Activate` and `Keep Inactive` branches
- fix the `updateClientDto={...}` contract
- keep final user-visible completion semantics stable
- explicitly test the protocol-vs-observed mismatch instead of normalizing it silently

Stage checks:
- Phase 8 uses explicit state branches
- activation DTO is authored correctly
- both terminal branches remain reachable and explicit

### Stage 6: Final consistency pass

Goals:
- run end-to-end trace across all rewritten files
- verify one consistent authoring style
- verify `ELSE IF` usage on mutually exclusive branches
- verify one consistent card-rendering verb
- verify every rewritten summary is self-contained

Stage checks:
- no legacy-only assumptions remain in rewritten files
- no duplicate phase definitions remain
- no phase-local behavior has drifted back into global files

## Acceptance and Measurement

### Functional acceptance

- Re-run the baseline flows from `delivery-rework/baseline-test-protocol.md`.
- Validate all three entry branches:
  - full setup
  - add-method
  - add-account
- Validate all major branch families:
  - schedule choice
  - all delivery types
  - mapping skip / enabled
  - content-type repair loops
  - connection-test success / failure / retry / skip
  - criteria skip / add / show-more / multi-criteria / enum
  - both activation choices

### Correctness acceptance

- Explicitly verify fixes for:
  - Phase 3 / 3b summary ownership
  - canonical Phase 6 only
  - multi-criteria preservation
  - FTP protocol variable usage
  - explicit connection-test capability state
  - `updateClientDto={...}` in activation

### Optimality acceptance

- On every stage, compare against the original `delivery/` implementation and the shared example patterns from this chat.
- End-to-end target:
  - no added happy-path assistant turns
  - no single baseline run regresses by more than 5% total AI time
  - target at least 10% average improvement across the 3 baseline runs
- Stage-specific emphasis:
  - Stage 2: eliminate Phase 2 summarize/reload retry loops
  - Stage 3: accept one additional resource hop only if it removes a branch-dense resource and makes branch tracing materially stronger
  - Stage 4: keep criteria behavior but reduce hidden state ambiguity and duplicated resource cost

## Notes for Future Sessions

- `delivery/` is the control copy; do not edit it during this migration.
- `delivery-rework/` is the only rewrite target.
- Before each new stage, start with:
  1. repo diff review
  2. branch trace
  3. cross-agent review
  4. optimality comparison against original + shared examples
- Treat this file as the canonical migration checklist unless it is explicitly replaced.
