# Baseline Test Results

## How This Report Was Produced

Session logs were saved as static HTML from the LeadExec admin UI (`/ai/administration/chat-details/view`). This report was extracted from those logs and then checked back against the original HTML. All 138 assistant message rows across the 3 sessions match the source.

Timings shown here are **AI Assistant response times only**. Tool execution times are excluded from the row-level tables and totals. For `get_resource`, the measured time is the assistant-side load and instruction-processing step; the paired Tool row that returns the resource body is untimed in these logs.

## Test Legend

| Label | Test ID | Session ID | Client |
|---|---|---|---|
| **BL-01** | Test 1 | `69cc18912697cf076414a968` | BMQAP01 / bmqap01@example.com |
| **BL-02** | Test 2 | `69cc1a8c2697cf076414a9b3` | BMQAP02 / bmqap02@example.com |
| **BL-03** | Test 3 | `69cc1d752697cf076414aa0f` | BMQAP03 / bmqap03@example.com |

## Log Links

- **BL-01**: https://dev.leadexec.app/App/#/ai/administration/chat-details/view/10/69cc18912697cf076414a968
- **BL-02**: https://dev.leadexec.app/App/#/ai/administration/chat-details/view/10/69cc1a8c2697cf076414a9b3
- **BL-03**: https://dev.leadexec.app/App/#/ai/administration/chat-details/view/10/69cc1d752697cf076414aa0f

## Sessions

| | BL-01 | BL-02 | BL-03 |
|---|---|---|---|
| Model | GPT-5 Mini | GPT-5 Mini | GPT-5 Mini |
| Messages (assistant / tool) | 45 / 39 | 51 / 45 | 42 / 36 |
| Tokens (in / out) | 318,504 / 27,541 | 321,878 / 36,497 | 276,586 / 32,170 |
| **AI Processing Time** | **6m 43s** | **9m 50s** | **7m 58s** |
| Anomaly | create_client failed, retried | Phase 2 looped 3x before progressing | None |

All tests completed the full flow. All ended with "Activate" (protocol specifies "Keep Inactive").

---

## Where Time Goes

Every assistant message falls into one of five categories. Response time = LLM inference time.

| Category | What it does | BL-01 | BL-02 | BL-03 | Avg % |
|---|---|---|---|---|---|
| **CARD** -- display_adaptive_card | LLM builds Adaptive Card JSON for UI | 3m 28s | 5m 12s | 4m 07s | **52%** |
| **API** -- LeadExec tool calls | LLM builds payload + server executes | 1m 17s | 2m 00s | 1m 27s | **19%** |
| **TEXT** -- plain text to user | LLM writes conversational response | 42s | 41s | 1m 01s | **10%** |
| **SUMMARY** -- summarize_history | AI Assistant prepares context summary; source logs show long pre-tool processing, while the tool call itself is only a few ms | 38s | 58s | 40s | **9%** |
| **LOAD+PROC** -- get_resource | AI Assistant loads the phase resource and processes returned instructions | 38s | 58s | 43s | **10%** |

Server-side API execution totals only 10-18s per session (2-4% of total); most time is assistant-side processing. In the source logs, `get_resource` time appears only on the assistant load-and-process call, not on the returned resource-body tool row. `summarize_history` shows the same split: the tool itself takes only 4-20 ms, while the visible multi-second delay sits on the assistant row before the call. The logs do not separate that assistant time into instruction reading, context parsing, reasoning, or summary generation.

---

## Per-Phase Timing

Each phase includes its entry `LOAD+PROC`, all interactions, and the exit `SUMMARY` into the next phase.

| Phase | What happens | BL-01 | BL-02 | BL-03 |
|---|---|---|---|---|
| **P0** | Flow entry, classify intent, initial conversation-start prompt/context processing | 8s | 10s | 15s |
| **P1** | Create client (name, email) | 58s | 31s | 40s |
| **P2** | Select lead type (BL-02 includes multiple selection attempts) | 29s | **134s*** | 41s |
| **P3** | Create delivery method (schedule, type, URL, mappings) | **129s** | **210s** | **124s** |
| **P3b** | Test connection (skipped) | 20s | 27s | 27s |
| **P4** | Delivery method summary card | 19s | 21s | 20s |
| **P5** | Create delivery account (price, exclusivity, states, criteria) | **96s** | **108s** | **160s** |
| **P6** | Delivery account summary card | 29s | 31s | 24s |
| **P7** | Client summary card | 10s | 12s | 9s |
| **P8** | Activate client | 5s | 6s | 18s |

*\*BL-02 P2: lead type selection repeated 3x due to input format mismatch. Clean P2 averages ~30-31s.*

*P1 note: this phase total includes phase load, first prompt, assistant-side `create_client` call preparation, and summary. In all 3 tests, the largest P1 contributor is the assistant `create_client` step, not the tool execution. BL-01 is further inflated by a failed first `create_client` call caused by missing `password`, followed by a retry.*

**Bottleneck phases: P3 and P5** -- together they consume 55-60% of session time, driven by complex CARD generation (field mapping preview, criteria fields).

---

## Top 20 Slowest Operations

| # | Time | Test | Phase | Type | What |
|---|---|---|---|---|---|
| 1 | **55.5s** | BL-02 | P3 | CARD | Field mapping preview |
| 2 | **51.8s** | BL-01 | P3 | CARD | Field mapping preview |
| 3 | **49.1s** | BL-02 | P3 | API | create_delivery_method |
| 4 | **45.8s** | BL-03 | P3 | CARD | Field mapping preview |
| 5 | **45.5s** | BL-03 | P5 | CARD | Criteria fields |
| 6 | 31.1s | BL-03 | P5 | TEXT | States targeting question |
| 7 | 29.6s | BL-02 | P5 | CARD | Criteria fields |
| 8 | 28.0s | BL-01 | P5 | CARD | Criteria fields |
| 9 | 26.5s | BL-02 | P2 | CARD | Lead type selector (retry 3) |
| 10 | 26.5s | BL-02 | P3 | CARD | mobile_phone mapping |
| 11 | 26.4s | BL-02 | P2 | CARD | Lead type selector (retry 2) |
| 12 | 25.9s | BL-03 | P2 | CARD | Lead type selector |
| 13 | 25.4s | BL-01 | P1 | API | create_client (failed) |
| 14 | 24.8s | BL-02 | P2 | CARD | Lead type selector |
| 15 | 22.9s | BL-02 | P6 | CARD | Delivery account summary |
| 16 | 22.7s | BL-03 | P5 | CARD | Exclusive/shared choice |
| 17 | 22.6s | BL-01 | P6 | CARD | Delivery account summary |
| 18 | 22.4s | BL-03 | P1 | API | create_client |
| 19 | 22.0s | BL-02 | P5 | API | get_usa_states |
| 20 | 21.3s | BL-02 | P1 | API | create_client |

15 of 20 are CARD, 4 are API, 1 is TEXT.

---

## BL-01 -- Session Log

Session `69cc18912697cf076414a968` | Client: BMQAP01 | Total AI time: **6m 43s** | 45 assistant messages

### P0 -- Flow Entry
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 1 | 8,452 | 2,578 | 571 | SUMMARY | summarize_history |

### P1 -- Create Client
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 2 | 4,373 | 2,659 | 102 | LOAD+PROC | phase-1-create-client |
| 3 | 6,511 | 3,219 | 428 | TEXT | "Provide company name, email" |
| 4 | **25,390** | 3,285 | 1,999 | API | create_client -- FAILED |
| 5 | 13,712 | 3,404 | 927 | API | create_client -- retry, success |
| 6 | 8,006 | 3,518 | 568 | SUMMARY | summarize_history >> P2 |

### P2 -- Select Lead Type
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 7 | 3,828 | 2,970 | 168 | LOAD+PROC | phase-2-get-lead-types |
| 8 | 1,128 | 2,988 | 20 | API | get_lead_types |
| 9 | 17,125 | 3,606 | 1,168 | CARD | Lead type selector |
| 10 | 7,116 | 4,513 | 415 | SUMMARY | summarize_history >> P3 |

### P3 -- Create Delivery Method
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 11 | 5,026 | 2,820 | 360 | LOAD+PROC | phase-3-create-delivery-method |
| 12 | 7,914 | 6,796 | 657 | CARD | Delivery schedule (24/7 vs specific) |
| 13 | 10,896 | 7,186 | 724 | CARD | Delivery type (Webhook/Portal/Email/FTP) |
| 14 | 4,880 | 7,425 | 277 | TEXT | "What's your webhook URL?" |
| 15 | 13,528 | 7,455 | 976 | CARD | Field mapping choice |
| 16 | 5,232 | 7,625 | 289 | API | get_lead_type (fetch fields) |
| 17 | 3,422 | 10,805 | 167 | CARD | Content type (JSON/XML/URL-encoded) |
| 18 | 1,631 | 11,184 | 51 | LOAD+PROC | phase-3 re-load |
| 19 | 4,774 | 14,256 | 277 | TEXT | "Paste JSON schema" |
| 20 | **51,790** | 14,723 | **4,302** | CARD | **Field mapping preview** |
| 21 | 14,388 | 16,551 | 1,076 | API | create_delivery_method |
| 22 | 5,414 | 17,653 | 374 | SUMMARY | summarize_history >> P3b |

### P3b -- Test Connection
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 23 | 3,673 | 4,008 | 232 | LOAD+PROC | phase-3b-test-connection |
| 24 | 9,127 | 3,863 | 731 | CARD | Test or skip connection |
| 25 | 6,777 | 4,077 | 384 | SUMMARY | summarize_history >> P4 |

### P4 -- Delivery Method Summary
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 26 | 6,216 | 3,332 | 424 | LOAD+PROC | phase-4-delivery-method-summary |
| 27 | 12,649 | 3,805 | 977 | CARD | Delivery method summary |

### P5 -- Create Delivery Account
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 28 | 3,050 | 4,357 | 168 | LOAD+PROC | phase-5-create-delivery-account |
| 29 | 15,719 | 7,379 | 986 | TEXT | "Provide price per lead" |
| 30 | 17,454 | 7,562 | 1,133 | CARD | Exclusive vs shared |
| 31 | 2,851 | 7,692 | 106 | CARD | Order system (yes/no) |
| 32 | 3,697 | 7,825 | 225 | API | get_lead_type (fetch fields) |
| 33 | 9,479 | 11,005 | 668 | TEXT | "Which states to target?" |
| 34 | **27,963** | 11,216 | **2,232** | CARD | **Criteria fields** |
| 35 | 8,857 | 11,421 | 410 | API | get_usa_states |
| 36 | 2,296 | 12,043 | 112 | CARD | State confirmation |
| 37 | 2,695 | 12,312 | 148 | API | create_delivery_account |
| 38 | 1,973 | 12,469 | 101 | SUMMARY | summarize_history >> P6 |

### P6 -- Delivery Account Summary
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 39 | 6,263 | 3,864 | 296 | LOAD+PROC | phase-6-delivery-account-summary |
| 40 | 22,624 | 4,002 | 1,531 | CARD | Delivery account summary |

### P7 -- Client Summary
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 41 | 2,010 | 4,673 | 102 | LOAD+PROC | phase-7-client-summary |
| 42 | 8,151 | 5,329 | 523 | CARD | Client summary |

### P8 -- Activation
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 43 | 1,795 | 5,941 | 38 | LOAD+PROC | phase-8-activation |
| 44 | 1,912 | 6,495 | 95 | API | update_client (activate) |
| 45 | 1,009 | 6,615 | 23 | TEXT | "Setup complete. ACTIVE." |

---

## BL-02 -- Session Log

Session `69cc1a8c2697cf076414a9b3` | Client: BMQAP02 | Total AI time: **9m 50s** | 51 assistant messages

### P0 -- Flow Entry
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 1 | 10,432 | 2,578 | 699 | SUMMARY | summarize_history |

### P1 -- Create Client
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 2 | 4,708 | 2,659 | 294 | LOAD+PROC | phase-1-create-client |
| 3 | 3,478 | 3,219 | 172 | TEXT | "Provide company name, email" |
| 4 | 21,312 | 3,285 | 1,376 | API | create_client |
| 5 | 1,989 | 3,400 | 114 | SUMMARY | summarize_history >> P2 |

### P2 -- Select Lead Type (3 attempts)
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 6 | 5,443 | 2,970 | 296 | LOAD+PROC | phase-2-get-lead-types |
| 7 | 999 | 2,988 | 20 | API | get_lead_types |
| 8 | 24,835 | 3,606 | 1,971 | CARD | Lead type selector (attempt 1) |
| 9 | 7,666 | 4,869 | 543 | SUMMARY | summarize_history (retry loop) |
| 10 | 5,292 | 2,820 | 360 | LOAD+PROC | phase-2 reload |
| 11 | 767 | 3,047 | 20 | API | get_lead_types |
| 12 | 26,438 | 3,665 | 1,544 | CARD | Lead type selector (attempt 2) |
| 13 | 8,314 | 4,558 | 415 | SUMMARY | summarize_history (retry loop) |
| 14 | 5,901 | 2,879 | 296 | LOAD+PROC | phase-2 reload |
| 15 | 9,982 | 3,106 | 538 | API | get_lead_types |
| 16 | 26,521 | 3,724 | 1,701 | CARD | Lead type selector (attempt 3) |
| 17 | 11,417 | 4,967 | 735 | SUMMARY | summarize_history >> P3 |

### P3 -- Create Delivery Method
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 18 | 7,661 | 2,938 | 488 | LOAD+PROC | phase-3-create-delivery-method |
| 19 | 15,846 | 6,914 | 714 | CARD | Delivery schedule (24/7 vs specific) |
| 20 | 10,663 | 7,293 | 704 | CARD | Delivery type (Webhook/Portal/Email/FTP) |
| 21 | 7,722 | 7,507 | 469 | TEXT | "What's your webhook URL?" |
| 22 | 14,046 | 7,537 | 906 | CARD | Field mapping choice |
| 23 | 6,623 | 7,698 | 353 | API | get_lead_type (fetch fields) |
| 24 | 4,909 | 10,878 | 282 | CARD | Content type (JSON/XML/URL-encoded) |
| 25 | 4,357 | 11,233 | 213 | TEXT | "Paste JSON schema" |
| 26 | 26,489 | 11,484 | 1,833 | CARD | mobile_phone field disambiguation |
| 27 | **55,505** | 11,675 | **3,652** | CARD | **Field mapping preview** |
| 28 | **49,099** | 13,172 | **3,404** | API | **create_delivery_method** |
| 29 | 7,384 | 14,163 | 354 | SUMMARY | summarize_history >> P3b |

### P3b -- Test Connection
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 30 | 5,417 | 4,113 | 232 | LOAD+PROC | phase-3b-test-connection |
| 31 | 12,863 | 3,968 | 850 | CARD | Test or skip connection |
| 32 | 8,284 | 4,237 | 512 | SUMMARY | summarize_history >> P4 |

### P4 -- Delivery Method Summary
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 33 | 5,837 | 3,437 | 360 | LOAD+PROC | phase-4-delivery-method-summary |
| 34 | 14,715 | 3,910 | 1,041 | CARD | Delivery method summary |

### P5 -- Create Delivery Account
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 35 | 3,707 | 4,462 | 168 | LOAD+PROC | phase-5-create-delivery-account |
| 36 | 10,708 | 7,484 | 602 | TEXT | "Provide price per lead" |
| 37 | 16,124 | 7,667 | 941 | CARD | Exclusive vs shared |
| 38 | 2,159 | 7,797 | 106 | CARD | Order system (yes/no) |
| 39 | 4,501 | 7,930 | 225 | API | get_lead_type (fetch fields) |
| 40 | 13,856 | 11,110 | 924 | TEXT | "Which states to target?" |
| 41 | **29,641** | 11,321 | **1,847** | CARD | **Criteria fields** |
| 42 | 21,987 | 11,525 | 1,370 | API | get_usa_states |
| 43 | 2,664 | 12,147 | 129 | API | create_delivery_account |
| 44 | 2,458 | 12,302 | 99 | SUMMARY | summarize_history >> P6 |

### P6 -- Delivery Account Summary
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 45 | 8,233 | 3,967 | 552 | LOAD+PROC | phase-6-delivery-account-summary |
| 46 | 22,857 | 4,105 | 1,274 | CARD | Delivery account summary |

### P7 -- Client Summary
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 47 | 3,460 | 4,775 | 102 | LOAD+PROC | phase-7-client-summary |
| 48 | 8,539 | 5,431 | 523 | CARD | Client summary |

### P8 -- Activation
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 49 | 2,787 | 6,043 | 38 | LOAD+PROC | phase-8-activation |
| 50 | 2,226 | 6,597 | 113 | API | update_client (activate) |
| 51 | 1,282 | 6,718 | 23 | TEXT | "Setup complete. ACTIVE." |

---

## BL-03 -- Session Log

Session `69cc1d752697cf076414aa0f` | Client: BMQAP03 | Total AI time: **7m 58s** | 42 assistant messages

### P0 -- Flow Entry
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 1 | 14,749 | 2,578 | 955 | SUMMARY | summarize_history |

### P1 -- Create Client
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 2 | 5,693 | 2,659 | 294 | LOAD+PROC | phase-1-create-client |
| 3 | 7,756 | 3,219 | 428 | TEXT | "Set up your Client..." |
| 4 | 22,355 | 3,285 | 1,374 | API | create_client |
| 5 | 4,252 | 3,398 | 114 | SUMMARY | summarize_history >> P2 |

### P2 -- Select Lead Type
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 6 | 5,863 | 2,970 | 296 | LOAD+PROC | phase-2-get-lead-types |
| 7 | 973 | 2,988 | 20 | API | get_lead_types |
| 8 | 25,931 | 3,606 | 1,558 | CARD | Lead type selector |
| 9 | 8,613 | 4,520 | 543 | SUMMARY | summarize_history >> P3 |

### P3 -- Create Delivery Method
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 10 | 5,597 | 2,820 | 360 | LOAD+PROC | phase-3-create-delivery-method |
| 11 | 11,129 | 6,796 | 714 | CARD | Delivery schedule (24/7 vs specific) |
| 12 | 16,498 | 7,175 | 1,344 | CARD | Delivery type (Webhook/Portal/Email/FTP) |
| 13 | 4,702 | 7,389 | 341 | TEXT | "What's your webhook URL?" |
| 14 | 7,225 | 7,419 | 650 | CARD | Field mapping choice |
| 15 | 3,398 | 7,580 | 161 | API | get_lead_type (fetch fields) |
| 16 | 5,431 | 10,760 | 346 | CARD | Content type (JSON/XML/URL-encoded) |
| 17 | 5,087 | 11,115 | 341 | TEXT | "Paste JSON schema" |
| 18 | **45,782** | 11,366 | **4,148** | CARD | **Field mapping preview** |
| 19 | 14,689 | 12,783 | 1,333 | API | create_delivery_method |
| 20 | 4,335 | 14,142 | 332 | SUMMARY | summarize_history >> P3b |

### P3b -- Test Connection
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 21 | 6,057 | 3,978 | 360 | LOAD+PROC | phase-3b-test-connection |
| 22 | 14,000 | 3,833 | 584 | CARD | Test or skip connection |
| 23 | 6,663 | 4,028 | 320 | SUMMARY | summarize_history >> P4 |

### P4 -- Delivery Method Summary
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 24 | 6,962 | 3,302 | 296 | LOAD+PROC | phase-4-delivery-method-summary |
| 25 | 13,501 | 3,775 | 849 | CARD | Delivery method summary |

### P5 -- Create Delivery Account
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 26 | 4,878 | 4,327 | 232 | LOAD+PROC | phase-5-create-delivery-account |
| 27 | 11,154 | 7,349 | 666 | TEXT | "Set up your Delivery Account" |
| 28 | 22,682 | 7,532 | 1,389 | CARD | Exclusive vs shared |
| 29 | 12,462 | 7,662 | 752 | CARD | Order system (yes/no) |
| 30 | 6,107 | 7,795 | 353 | API | get_lead_type (fetch fields) |
| 31 | **31,140** | 10,975 | **2,012** | TEXT | **States targeting question** |
| 32 | **45,454** | 11,186 | **3,256** | CARD | **Criteria fields** |
| 33 | 21,249 | 11,391 | 1,562 | API | get_usa_states |
| 34 | 2,691 | 12,013 | 148 | API | create_delivery_account |
| 35 | 1,789 | 12,170 | 101 | SUMMARY | summarize_history >> P6 |

### P6 -- Delivery Account Summary
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 36 | 4,842 | 3,834 | 232 | LOAD+PROC | phase-6-delivery-account-summary |
| 37 | 19,272 | 3,972 | 1,595 | CARD | Delivery account summary |

### P7 -- Client Summary
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 38 | 1,782 | 4,642 | 102 | LOAD+PROC | phase-7-client-summary |
| 39 | 7,625 | 5,298 | 522 | CARD | Client summary |

### P8 -- Activation
| # | Response Time (ms) | Tokens In | Tokens Out | Type | Operation |
|---|---|---|---|---|---|
| 40 | 1,776 | 5,909 | 38 | LOAD+PROC | phase-8-activation |
| 41 | 15,114 | 6,463 | 1,126 | API | update_client (activate) |
| 42 | 868 | 6,584 | 23 | TEXT | "Setup complete. ACTIVE." |

---

## Key Findings

1. **CARD generation is the #1 bottleneck** -- ~52% of all processing time across all tests. This is pure LLM inference to produce Adaptive Card JSON.

2. **Two specific cards dominate** -- Field mapping preview (46-56s, ~4,000 output tokens) and Criteria fields (28-45s, ~2,500 output tokens) are consistently the slowest operations.

3. **Output tokens drive CARD time** -- strong correlation between output token count and response time. High-token cards are the optimization target.

4. **Actual API calls are fast** -- server-side execution totals 10-18s per session (2-4%). The bottleneck is LLM inference, not backend.

5. **Phase transitions cost ~9-10%** -- each `SUMMARY` + `LOAD+PROC` pair averages ~11s. With 6-8 transitions per session, this adds 66-116s of overhead.

6. **P3 and P5 are the slowest phases** -- together they consume 55-60% of total session time.
