# Delivery Rework — Architecture

## File Structure

```
system/rw-1-global.md                          Global system instructions (~72 lines)

action/rw-create-single-client.md              full-setup → Phase 1
action/rw-create-delivery-method.md            add-method → Phase 1a
action/rw-create-delivery-account.md           add-account → Phase 1b

resources/rw-phase-1-create-client.md          Create new client
resources/rw-phase-1a-select-client.md         Select existing client (add-method)
resources/rw-phase-1b-select-client-and-method.md  Select client + method (add-account)
resources/rw-phase-2-get-lead-types.md         Lead type selection
resources/rw-phase-3-create-delivery-method.md Router: schedule + type → per-type resource
resources/rw-phase-3-portal.md                 Portal: create → Phase 4 (no test)
resources/rw-phase-3-email.md                  Email: create → Phase 4 (no test)
resources/rw-phase-3-ftp.md                    FTP: collect creds + create → Phase 3a
resources/rw-phase-3-webhook.md                Webhook: URL + mapping + create → Phase 3b
resources/rw-phase-3a-ftp-test.md              FTP connection test → Phase 4
resources/rw-phase-3b-webhook-test.md          Webhook connection test → Phase 4
resources/rw-phase-4-delivery-method-summary.md Summary card → Phase 5 or done
resources/rw-phase-5-create-delivery-account.md Create account (state criteria) → 5c or Phase 6
resources/rw-phase-5c-criteria-builder.md      Criteria loop → update account → Phase 6
resources/rw-phase-6-delivery-account-summary.md Summary card → Phase 7 or done
resources/rw-phase-7-client-summary.md         Final summary → Phase 8
resources/rw-phase-8-activation.md             Activate or keep inactive (terminal)
```

## Flow Graph

```
full-setup:
  Phase 1 → 2 → 3 router → {portal|email|ftp|webhook}
    portal/email → Phase 4
    ftp → 3a test → Phase 4
    webhook → 3b test → Phase 4
  Phase 4 → 5 (create account) → "criteria?"
    skip → Phase 6
    add → 5c (update account) → Phase 6
  Phase 6 → 7 → 8 (terminal)

add-method:
  Phase 1a → 2 → 3 router → {type} → [test] → Phase 4 (done)

add-account:
  Phase 1b → 5 → [5c] → Phase 6 (done)
```

## Key Design Decisions

- **Single anchor** (`DELIVERY_SETUP_START`) in action system message. Persists across all summarizations.
- **Bullet-list summaries** with `{variable}` placeholders. One field per line — no comma fragility.
- **Per-type delivery resources** — Portal, Email, FTP, Webhook each have their own file. Only relevant logic is loaded.
- **Per-type connection tests** — 3a (FTP) and 3b (Webhook). Portal/Email skip testing entirely.
- **Create first, update later** — Phase 5 creates account with state-only criteria. Phase 5c uses `update_delivery_account` to add additional criteria.
- **Rules inline** — normalization, operator tables, field matching rules appear inside the state that uses them, not at top of file.
- **Simplified actions** — no Phase 0 router, no intent detection. Each action sets flowIntent directly.
- **Lean imperative format** — CRITICAL STATE UPDATE header, explicit STOP AND YIELD with "Do not hallucinate data", minimal states per resource.

## Bugs Fixed vs Original (13)

1. Multi-criteria array preservation (append, not overwrite)
2. updateClientDto nested correctly (not flat fields)
3. FTP protocol variable (protocol, not detectedProtocol)
4. connectionTestMode flag replaces URL heuristic
5. Canonical Phase 6 (removed duplicate from Phase 5)
6. Phase 2 summary carries all prior state (was losing flowIntent, clientUID)
7. Phase 3b summary carries all Phase 3 state (was minimal)
8. Phase 5 summary carries delivery-method fields (was dropping them)
9. Phase 6 boolean display (Exclusive/Shared, not true/false)
10. leadFieldsMap includes specialBit for operator validation
11. Enum operator forced to In/NotIn before retention
12. Enum values resolve to leadFieldEnumUID (not raw text)
13. Empty list handling in Phase 1a/1b

## Verification

Verified with 70+ agentic tests across 9 rounds covering all 3 entry flows, all 4 delivery types, repair paths, multi-criteria with enums, content type switching, XML/JSON/URL-Encoded mapping, connection test success/fail/retry/skip, and activation with updateClientDto overwrite semantics.
