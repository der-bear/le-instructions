# Delivery Minimal — UX Scenarios

## Scenario 1 — Full Setup, Portal, All States
1. User starts full setup.
2. Agent creates the client.
3. Agent shows a `Client created` adaptive-card table and waits for `Continue to Lead Type`.
4. Agent shows lead-type selection.
5. Agent uses the Phase 3 router to collect `Portal`.
6. Agent routes to the Portal method resource, collects schedule there, and creates the Portal method.
7. Agent shows a `Method created` adaptive-card table and waits for `Continue to Delivery Account`.
8. Agent asks account questions in the fixed order.
9. User chooses `All states` and no extra criteria.
10. Agent creates the account once.
11. Agent shows an `Account created` adaptive-card table and waits for `Continue to Activation`.
12. Agent shows a short plain-text setup summary, offers `Activate now` or `Keep inactive`, and the user keeps the client inactive.

## Scenario 2 — Full Setup, Webhook, Skip Mapping
1. User selects `Webhook`.
2. Agent collects schedule inside the Webhook resource.
3. Agent asks for endpoint URL.
4. Agent asks `Provide instructions` or `Skip mapping`.
5. User chooses `Skip mapping`.
6. Agent creates the webhook method with default field names.
7. Agent shows a post-create `Method created` adaptive-card table.
8. After `Continue to Delivery Account`, the flow moves to account creation.

## Scenario 3 — Full Setup, Webhook, Explicit JSON Spec
1. User selects `Webhook`.
2. Agent collects schedule inside the Webhook resource.
3. User provides URL and chooses `Provide instructions`.
4. User explicitly selects `JSON`.
5. User pastes a JSON schema.
6. Agent loads lead fields only after the schema is accepted.
7. Agent builds mappings and shows an adaptive-card table review with mapped field pairs.
8. Agent asks for final confirmation and creates the webhook method only after `Create method`.
9. Agent shows a post-create `Method created` adaptive-card table before moving to the next phase.

## Scenario 4 — Full Setup, Webhook, Auto-Detect Content Type
1. User selects `Webhook`.
2. Agent collects schedule inside the Webhook resource.
3. User provides URL and chooses `Provide instructions`.
4. User selects `I'm not sure`.
5. User pastes a usable JSON or XML body.
6. Agent detects the format once, rewrites the working content type to that detected value, confirms it once, and continues normally.

## Scenario 5 — Full Setup, Webhook, Large Spec
1. User pastes a very large or messy spec.
2. Agent does not try to process everything blindly.
3. Agent asks for a smaller excerpt or offers `Skip mapping`.
4. Flow continues from the user's choice without stalling.

## Scenario 6 — Method-Only Flow
1. User starts add-method flow.
2. Agent selects an existing client.
3. Agent selects a lead type.
4. Agent routes through the Phase 3 router to the selected simple-method resource such as Portal or Email.
5. Agent collects schedule inside that method resource.
6. Agent shows a `Method created` adaptive-card table.
7. Agent waits for `Finish`.
8. Agent replies with the new `deliveryMethodUID` and stops.

## Scenario 7 — Account-Only Flow
1. User starts add-account flow.
2. Agent selects an existing client.
3. Agent selects an existing delivery method and retains `leadTypeUID`.
4. Agent asks the fixed account questions.
5. Agent creates the account once.
6. Agent shows an `Account created` adaptive-card table.
7. Agent waits for `Finish`.
8. Agent replies with the new `deliveryAccountUID` and stops.

## Scenario 8 — Typed Fallback Everywhere
1. Every card selection is answered by typing text instead of clicking.
2. The agent matches typed values safely.
3. Ambiguous typed values trigger a focused retry, not a phase reset.

## Scenario 9 — State Ambiguity
1. The selected lead type exposes more than one plausible state field.
2. The agent asks one focused fallback question with candidate field names.
3. The agent retains the pending candidates and resolves `stateFieldUID` from the user's reply.
4. If ambiguity remains, the agent offers `continue without state targeting` or `cancel`.

## Scenario 10 — Criteria Compile Failure
1. User requests extra criteria.
2. The criteria builder cannot safely compile a criterion.
3. The agent offers only:
   - `Fix criteria`
   - `Continue with geography only`
4. `Fix criteria` returns to the criteria builder instead of stalling in Phase 4.
5. `Continue with geography only` clears extra criteria and proceeds with geography-only account creation.

## Scenario 11 — Webhook Review Revisions
1. User provides a valid webhook spec and reaches the adaptive-card table review.
2. The agent offers only:
   - `Create method`
   - `Revise URL`
   - `Revise spec`
   - `Change content type`
   - `Skip mapping`
3. Each revision choice resets only the intended webhook inputs.
4. The method is not created until the user explicitly chooses `Create method`.
5. After creation succeeds, the flow still shows the post-create `Method created` adaptive-card table before handoff.

## Scenario 12 — Malformed JSON Repair
1. User pastes JSON with missing outer braces, a trailing comma, and single quotes.
2. The agent applies one lossless repair pass.
3. If the repaired spec is usable, the flow continues without forcing a re-paste.
4. If it is still unusable, the agent offers only:
   - `Re-paste excerpt`
   - `Change content type`
   - `Skip mapping`
5. `Re-paste excerpt` returns to the posting-instructions step while keeping the accepted URL and current content type.

## Scenario 13 — Webhook Ambiguity Clarification
1. The webhook spec yields at least one ambiguous delivery field mapping.
2. The agent asks for exactly one delivery field clarification at a time.
3. The user resolves or skips that one field.
4. The agent recomputes counts and unresolved fields before showing the final review card.

## Scenario 14 — Partial State Match Rejected
1. User selects `Specific states`.
2. User enters one valid state and one invalid state.
3. The agent rejects the full input instead of silently keeping the valid subset.
4. The agent offers only:
   - `Re-enter states`
   - `All states`

## Scenario 15 — Enum Criterion Exit
1. User requests extra criteria and names an enumerated field without a value.
2. The agent shows a compact ChoiceSet plus Submit with no extra submit data.
3. If the user types `continue`, the builder exits cleanly with the criteria collected so far.
4. If the user types `continue with geography only`, the builder clears all compiled criteria and returns to Phase 4.

## Scenario 16 — Criteria Builder Return To Phase 4
1. User answers `Yes` to extra criteria.
2. Phase 4 immediately loads Phase 4b and stops instead of continuing into account creation.
3. The user finishes criteria entry with `done`.
4. Phase 4 resumes once, builds the final `criteriaPayload`, and creates the account once.

## Scenario 17 — Activation Failure Recovery
1. The user chooses `Activate now`.
2. The activation call fails.
3. The agent offers only:
   - `Retry activation`
   - `Keep inactive`
4. Retry resends the full client DTO with a fresh password.

## Scenario 18 — Repeated Activation Failure Recovery
1. The user chooses `Activate now`.
2. The activation call fails twice in a row.
3. After each failure, the agent offers only `Retry activation` and `Keep inactive`.
4. The retry path never falls back to the initial setup-summary card before handling the retry.

## Scenario 19 — Preview Acknowledgement Does Not Re-Create
1. The agent creates a client, method, or account and shows the corresponding created-state preview.
2. The user clicks the acknowledgement action or types the same acknowledgement.
3. The agent does not re-run the create mutation.
4. The agent summarizes or completes only after handling the acknowledgement.

## Scenario 20 — FTP Preview Redaction
1. User creates an FTP delivery method.
2. The post-create `Method created` preview includes the host and username.
3. The FTP password is shown only as `set`.

## Scenario 21 — Normalized Preview Values
1. The user creates an Email method and an account with `Shared`, `Yes`, and specific states entered in mixed formats.
2. The method preview shows `Email`, not `EMail`.
3. The account preview shows `Shared`, `Yes`, `$NN.NN`, and `Specific states: <normalized state codes>`.
4. No preview table shows raw booleans or raw DTO enum labels.

## Scenario 22 — Webhook Review Size Guard
1. The user provides a webhook spec that would produce too many mapped rows or an oversized unresolved-or-omitted list.
2. The agent does not render a giant mapping-review table.
3. The agent offers only `Revise spec` or `Skip mapping`.

## Scenario 23 — Activation Order Note
1. The user creates an account with `useOrder=true`.
2. The activation phase shows the standard plain-text setup summary before the final choice.
3. The order follow-up note appears in that summary.
4. The same note does not appear when `useOrder=false`.

## Scenario 24 — Phase 3 Branch Reset
1. The user enters Phase 3, chooses one method type, and partially fills that branch.
2. The user restarts Phase 3 and chooses a different method type.
3. The router clears stale Phase 3 method-attempt state before loading the new branch.
4. The new branch asks for its own schedule and branch-specific inputs instead of silently reusing the earlier branch's values.

## Scenario 24 — Card Render Retry And Fallback
1. A required preview or webhook review card fails to render.
2. The agent retries the same card once immediately.
3. If the retry also fails, the agent uses one constrained plain-text fallback with the same key information and allowed actions.
4. The agent does not send both card and plain text in the same turn.
