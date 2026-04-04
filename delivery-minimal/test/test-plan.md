# Delivery Minimal — Test Plan

## Goal
Validate that the new pack keeps the current platform-facing architecture while removing the brittle summary/state-transfer behavior seen in the other variants.

## Required Scenarios
1. Full setup, Portal, 24/7, all states, keep inactive.
2. Full setup, Portal, specific states, no extra criteria.
3. Full setup, Email, activate at end.
4. Full setup, FTP, no extra criteria.
5. Full setup, Webhook, skip mappings.
6. Full setup, Webhook, JSON schema, auto-detect content type.
7. Full setup, Webhook, large or messy spec prompts for a smaller excerpt.
8. Method-only flow for an existing client.
9. Account-only flow for an existing client and method.
10. Typed-text fallback at every card step instead of button clicks.
11. State-field ambiguity fallback.
12. Extra criteria compile failure offers `fix` or `continue geography-only`.

## Acceptance Criteria
- No repeated intro prompts.
- No stall after lead type selection.
- `summarize_history` is used only for scalar phase handoffs:
  - Phase 0 -> 2
  - Phase 1 -> 2
  - Phase 2 -> 3
  - Phase 3 -> 4 for created simple methods
  - Phase 3 Webhook -> 4
  - Phase 4 -> 5
- No summary-review phases or "Continue" summary gates.
- No summary-card gating.
- No table-card dependency.
- No hidden required tool steps between user prompts.
- `deliveryMethodUID` is available when creating the account.
- Geography is always collected before account creation.
- Existing-client entry actions do not load irrelevant phases.
- The criteria builder never mutates the account directly.

## Explicit Checks
### Core Flow Checks
- `create-single-client` goes:
  - create client
  - select lead type
  - create delivery method
  - create delivery account
  - activation
- Scalar phase context is transferred via summaries rather than relying only on raw conversation history.
- `create-delivery-method` ends after the method is created.
- `create-delivery-account` ends after the account is created.

### Webhook Checks
- `get_lead_type` is not called until after the user has pasted a usable spec.
- URL-only specs are rejected.
- Large or repetitive specs trigger `smaller excerpt` or `skip mapping`.
- No mapping preview table is required.
- No connection-test phase is loaded.
- The router does not summarize before loading the webhook phase because `deliveryDays` must stay native.

### Account Checks
- The account flow asks in this exact order:
  1. price
  2. exclusivity
  3. use order
  4. geography choice
  5. states when needed
  6. extra criteria yes/no
- `All states` creates no state criterion.
- Extra criteria are compiled before `create_delivery_account`.
- The account is created once in the normal path.
- The criteria builder does not summarize before returning to Phase 4 because `compiledCriteria` must stay native.

## Defaults To Verify
- `dayMax=50`
- `hourMax=-1`
- `weekMax=-1`
- `monthMax=-1`
- `automationEnabled=true`
- account `status="Open"`
- activation only on explicit confirmation
