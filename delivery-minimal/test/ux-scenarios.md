# Delivery Minimal — UX Scenarios

## Scenario 1 — Full Setup, Portal, All States
1. User starts full setup.
2. Agent creates the client.
3. Agent shows lead-type selection.
4. Agent collects schedule and `Portal`.
5. Agent creates the Portal method with no summary gate.
6. Agent asks account questions in the fixed order.
7. User chooses `All states` and no extra criteria.
8. Agent creates the account once.
9. Agent shows plain-text summary and the user keeps the client inactive.

## Scenario 2 — Full Setup, Webhook, Skip Mapping
1. User selects `Webhook`.
2. Agent asks for endpoint URL.
3. Agent asks `Provide instructions` or `Skip mapping`.
4. User chooses `Skip mapping`.
5. Agent creates the webhook method immediately with default field names.
6. Agent moves directly to account creation.

## Scenario 3 — Full Setup, Webhook, JSON Spec
1. User selects `Webhook`.
2. User provides URL and chooses `Provide instructions`.
3. User pastes a JSON schema.
4. Agent confirms detected content type if needed.
5. Agent loads lead fields only after the schema is accepted.
6. Agent builds mappings and a plain-text mapping summary.
7. Agent asks for final confirmation and creates the webhook method.

## Scenario 4 — Full Setup, Webhook, Large Spec
1. User pastes a very large or messy spec.
2. Agent does not try to process everything blindly.
3. Agent asks for a smaller excerpt or offers `Skip mapping`.
4. Flow continues from the user's choice without stalling.

## Scenario 5 — Method-Only Flow
1. User starts add-method flow.
2. Agent selects an existing client.
3. Agent selects a lead type.
4. Agent creates the method.
5. Agent replies with the new `deliveryMethodUID` and stops.

## Scenario 6 — Account-Only Flow
1. User starts add-account flow.
2. Agent selects an existing client.
3. Agent selects an existing delivery method and retains `leadTypeUID`.
4. Agent asks the fixed account questions.
5. Agent creates the account once.
6. Agent replies with the new `deliveryAccountUID` and stops.

## Scenario 7 — Typed Fallback Everywhere
1. Every card selection is answered by typing text instead of clicking.
2. The agent matches typed values safely.
3. Ambiguous typed values trigger a focused retry, not a phase reset.

## Scenario 8 — State Ambiguity
1. The selected lead type exposes more than one plausible state field.
2. The agent asks one focused fallback question with candidate field names.
3. If ambiguity remains, the agent offers `continue without state targeting` or `cancel`.

## Scenario 9 — Criteria Compile Failure
1. User requests extra criteria.
2. The criteria builder cannot safely compile a criterion.
3. The agent offers only:
   - `Fix criteria`
   - `Continue with geography only`
4. The builder never mutates the account.
