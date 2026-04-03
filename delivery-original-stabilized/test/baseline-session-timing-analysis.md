# Delivery Baseline Session Timing Analysis

Double-checked against the original saved HTML logs. This report uses the row type shown in the log itself: `AI Assistant` rows and `Tool` rows.

## Log Parsing Rules

- `AIResourcesPlugin.get_resource` appears first as an `AI Assistant` function-call row with its own `Response time` badge.
- The following `Tool` row contains the returned resource body. Many of those tool rows have no visible `Response time` badge, so they remain `N/A`.
- `display_adaptive_card` and `summarize_history` follow the same pattern: assistant call row first, tool output row second.
- `Seq` values below are the original visible conversation row numbers from the saved HTML.
- All timings below are taken only from visible `Response time` badges in the original HTML.

## Test Summary

| Test | Total Processing Time | Min Processed Time Output | Max Processed Time Output |
|---|---:|---:|---:|
| Test 1 | 420,434 ms | 1 ms | 51,790 ms |
| Test 2 | 601,774 ms | 1 ms | 55,505 ms |
| Test 3 | 488,596 ms | 1 ms | 45,782 ms |

## Display Adaptive Card Review

| Test | AI Assistant `display_adaptive_card` Calls | Min AI Assistant Delay | Max AI Assistant Delay | Timed Tool Rows | Untimed Tool Rows |
|---|---:|---:|---:|---:|---:|
| Test 1 | 14 | 2,296 ms | 51,790 ms | 2 | 12 |
| Test 2 | 16 | 2,159 ms | 55,505 ms | 1 | 15 |
| Test 3 | 13 | 5,431 ms | 45,782 ms | 1 | 12 |

## Validation

| Test | Assistant Rows | Tool Rows | Timed Assistant Rows | Timed Tool Rows |
|---|---:|---:|---:|---:|
| Test 1 | 45 | 39 | 45 | 17 |
| Test 2 | 51 | 45 | 51 | 19 |
| Test 3 | 42 | 36 | 42 | 15 |

## Top 20 Max Delays Across Tests

| Rank | Test | Seq | Flow Phase | Row Type | Function / Label | Response Time |
|---:|---|---:|---|---|---|---:|
| 1 | Test 2 | 70 | Phase 3 Create Delivery Method | AI Assistant | DisplayPlugin.display_adaptive_card | 55,505 ms |
| 2 | Test 1 | 51 | Phase 3 Create Delivery Method | AI Assistant | DisplayPlugin.display_adaptive_card | 51,790 ms |
| 3 | Test 2 | 73 | Phase 3 Create Delivery Method | AI Assistant | LeadExecTools.create_delivery_method | 49,099 ms |
| 4 | Test 3 | 47 | Phase 3 Create Delivery Method | AI Assistant | DisplayPlugin.display_adaptive_card | 45,782 ms |
| 5 | Test 3 | 82 | Phase 5 Create Delivery Account | AI Assistant | DisplayPlugin.display_adaptive_card | 45,454 ms |
| 6 | Test 3 | 80 | Phase 5 Create Delivery Account | AI Assistant | assistant_response: Which states do you want to target? (e.g., CA, AZ, TX) | 31,140 ms |
| 7 | Test 2 | 105 | Phase 5 Create Delivery Account | AI Assistant | DisplayPlugin.display_adaptive_card | 29,641 ms |
| 8 | Test 1 | 86 | Phase 5 Create Delivery Account | AI Assistant | DisplayPlugin.display_adaptive_card | 27,963 ms |
| 9 | Test 2 | 41 | Phase 2 Get Lead Types | AI Assistant | DisplayPlugin.display_adaptive_card | 26,521 ms |
| 10 | Test 2 | 67 | Phase 3 Create Delivery Method | AI Assistant | DisplayPlugin.display_adaptive_card | 26,489 ms |
| 11 | Test 2 | 31 | Phase 2 Get Lead Types | AI Assistant | DisplayPlugin.display_adaptive_card | 26,438 ms |
| 12 | Test 3 | 21 | Phase 2 Get Lead Types | AI Assistant | DisplayPlugin.display_adaptive_card | 25,931 ms |
| 13 | Test 1 | 12 | Phase 1 Create Client | AI Assistant | LeadExecTools.create_client | 25,390 ms |
| 14 | Test 2 | 21 | Phase 2 Get Lead Types | AI Assistant | DisplayPlugin.display_adaptive_card | 24,835 ms |
| 15 | Test 2 | 117 | Phase 6 Delivery Account Summary | AI Assistant | DisplayPlugin.display_adaptive_card | 22,857 ms |
| 16 | Test 3 | 72 | Phase 5 Create Delivery Account | AI Assistant | DisplayPlugin.display_adaptive_card | 22,682 ms |
| 17 | Test 1 | 101 | Phase 6 Delivery Account Summary | AI Assistant | DisplayPlugin.display_adaptive_card | 22,624 ms |
| 18 | Test 3 | 12 | Phase 1 Create Client | AI Assistant | LeadExecTools.create_client | 22,355 ms |
| 19 | Test 2 | 108 | Phase 5 Create Delivery Account | AI Assistant | LeadExecTools.get_usa_states | 21,987 ms |
| 20 | Test 2 | 12 | Phase 1 Create Client | AI Assistant | LeadExecTools.create_client | 21,312 ms |

## Test 1

- Session ID: `69cc18912697cf076414a968`
- Client Username: `bmqap01@example.com`

| Seq | Flow Phase | Row Type | Function / Label | Response Time |
|---:|---|---|---|---:|
| 5 | Flow Start | AI Assistant | SummaryPlugin.summarize_history | 8,452 ms |
| 6 | Flow Start | Tool | SummaryPlugin.summarize_history | 11 ms |
| 8 | Phase 1 Create Client | AI Assistant | AIResourcesPlugin.get_resource | 4,373 ms |
| 9 | Phase 1 Create Client | Tool | AIResourcesPlugin.get_resource | N/A |
| 10 | Phase 1 Create Client | AI Assistant | assistant_response: Great - first we'll set up your Client, Delivery Method, and Delivery Account. T... | 6,511 ms |
| 12 | Phase 1 Create Client | AI Assistant | LeadExecTools.create_client | 25,390 ms |
| 13 | Phase 1 Create Client | Tool | LeadExecTools.create_client | 4,687 ms |
| 14 | Phase 1 Create Client | AI Assistant | LeadExecTools.create_client | 13,712 ms |
| 15 | Phase 1 Create Client | Tool | LeadExecTools.create_client | 2,417 ms |
| 16 | Phase 1 Create Client | AI Assistant | SummaryPlugin.summarize_history | 8,006 ms |
| 17 | Phase 1 Create Client | Tool | SummaryPlugin.summarize_history | 8 ms |
| 19 | Phase 2 Get Lead Types | AI Assistant | AIResourcesPlugin.get_resource | 3,828 ms |
| 20 | Phase 2 Get Lead Types | Tool | AIResourcesPlugin.get_resource | N/A |
| 21 | Phase 2 Get Lead Types | AI Assistant | LeadExecTools.get_lead_types | 1,128 ms |
| 22 | Phase 2 Get Lead Types | Tool | LeadExecTools.get_lead_types | 258 ms |
| 23 | Phase 2 Get Lead Types | AI Assistant | DisplayPlugin.display_adaptive_card | 17,125 ms |
| 24 | Phase 2 Get Lead Types | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 26 | Phase 2 Get Lead Types | AI Assistant | SummaryPlugin.summarize_history | 7,116 ms |
| 27 | Phase 2 Get Lead Types | Tool | SummaryPlugin.summarize_history | 9 ms |
| 29 | Phase 3 Create Delivery Method | AI Assistant | AIResourcesPlugin.get_resource | 5,026 ms |
| 30 | Phase 3 Create Delivery Method | Tool | AIResourcesPlugin.get_resource | N/A |
| 31 | Phase 3 Create Delivery Method | AI Assistant | DisplayPlugin.display_adaptive_card | 7,914 ms |
| 32 | Phase 3 Create Delivery Method | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 34 | Phase 3 Create Delivery Method | AI Assistant | DisplayPlugin.display_adaptive_card | 10,896 ms |
| 35 | Phase 3 Create Delivery Method | Tool | DisplayPlugin.display_adaptive_card | 2 ms |
| 37 | Phase 3 Create Delivery Method | AI Assistant | assistant_response: What's your webhook URL where we should send the leads? | 4,880 ms |
| 39 | Phase 3 Create Delivery Method | AI Assistant | DisplayPlugin.display_adaptive_card | 13,528 ms |
| 40 | Phase 3 Create Delivery Method | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 42 | Phase 3 Create Delivery Method | AI Assistant | LeadExecTools.get_lead_type | 5,232 ms |
| 43 | Phase 3 Create Delivery Method | Tool | LeadExecTools.get_lead_type | 2,866 ms |
| 44 | Phase 3 Create Delivery Method | AI Assistant | DisplayPlugin.display_adaptive_card | 3,422 ms |
| 45 | Phase 3 Create Delivery Method | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 47 | Phase 3 Create Delivery Method | AI Assistant | AIResourcesPlugin.get_resource | 1,631 ms |
| 48 | Phase 3 Create Delivery Method | Tool | AIResourcesPlugin.get_resource | N/A |
| 49 | Phase 3 Create Delivery Method | AI Assistant | assistant_response: Please paste the JSON schema that your client's API expects. | 4,774 ms |
| 51 | Phase 3 Create Delivery Method | AI Assistant | DisplayPlugin.display_adaptive_card | 51,790 ms |
| 52 | Phase 3 Create Delivery Method | Tool | DisplayPlugin.display_adaptive_card | 1 ms |
| 54 | Phase 3 Create Delivery Method | AI Assistant | LeadExecTools.create_delivery_method | 14,388 ms |
| 55 | Phase 3 Create Delivery Method | Tool | LeadExecTools.create_delivery_method | 2,396 ms |
| 56 | Phase 3 Create Delivery Method | AI Assistant | SummaryPlugin.summarize_history | 5,414 ms |
| 57 | Phase 3 Create Delivery Method | Tool | SummaryPlugin.summarize_history | 13 ms |
| 59 | Phase 3b Test Connection | AI Assistant | AIResourcesPlugin.get_resource | 3,673 ms |
| 60 | Phase 3b Test Connection | Tool | AIResourcesPlugin.get_resource | N/A |
| 61 | Phase 3b Test Connection | AI Assistant | DisplayPlugin.display_adaptive_card | 9,127 ms |
| 62 | Phase 3b Test Connection | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 64 | Phase 3b Test Connection | AI Assistant | SummaryPlugin.summarize_history | 6,777 ms |
| 65 | Phase 3b Test Connection | Tool | SummaryPlugin.summarize_history | 12 ms |
| 67 | Phase 4 Delivery Method Summary | AI Assistant | AIResourcesPlugin.get_resource | 6,216 ms |
| 68 | Phase 4 Delivery Method Summary | Tool | AIResourcesPlugin.get_resource | N/A |
| 69 | Phase 4 Delivery Method Summary | AI Assistant | DisplayPlugin.display_adaptive_card | 12,649 ms |
| 70 | Phase 4 Delivery Method Summary | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 72 | Phase 5 Create Delivery Account | AI Assistant | AIResourcesPlugin.get_resource | 3,050 ms |
| 73 | Phase 5 Create Delivery Account | Tool | AIResourcesPlugin.get_resource | N/A |
| 74 | Phase 5 Create Delivery Account | AI Assistant | assistant_response: Finally, let's set up your Delivery Account. Please provide the price per lead. | 15,719 ms |
| 76 | Phase 5 Create Delivery Account | AI Assistant | DisplayPlugin.display_adaptive_card | 17,454 ms |
| 77 | Phase 5 Create Delivery Account | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 79 | Phase 5 Create Delivery Account | AI Assistant | DisplayPlugin.display_adaptive_card | 2,851 ms |
| 80 | Phase 5 Create Delivery Account | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 82 | Phase 5 Create Delivery Account | AI Assistant | LeadExecTools.get_lead_type | 3,697 ms |
| 83 | Phase 5 Create Delivery Account | Tool | LeadExecTools.get_lead_type | 2,193 ms |
| 84 | Phase 5 Create Delivery Account | AI Assistant | assistant_response: Which states do you want to target? (e.g., CA, AZ, TX) | 9,479 ms |
| 86 | Phase 5 Create Delivery Account | AI Assistant | DisplayPlugin.display_adaptive_card | 27,963 ms |
| 87 | Phase 5 Create Delivery Account | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 89 | Phase 5 Create Delivery Account | AI Assistant | LeadExecTools.get_usa_states | 8,857 ms |
| 90 | Phase 5 Create Delivery Account | Tool | LeadExecTools.get_usa_states | 453 ms |
| 91 | Phase 5 Create Delivery Account | AI Assistant | DisplayPlugin.display_adaptive_card | 2,296 ms |
| 92 | Phase 5 Create Delivery Account | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 94 | Phase 5 Create Delivery Account | AI Assistant | LeadExecTools.create_delivery_account | 2,695 ms |
| 95 | Phase 5 Create Delivery Account | Tool | LeadExecTools.create_delivery_account | 1,588 ms |
| 96 | Phase 5 Create Delivery Account | AI Assistant | SummaryPlugin.summarize_history | 1,973 ms |
| 97 | Phase 5 Create Delivery Account | Tool | SummaryPlugin.summarize_history | 17 ms |
| 99 | Phase 6 Delivery Account Summary | AI Assistant | AIResourcesPlugin.get_resource | 6,263 ms |
| 100 | Phase 6 Delivery Account Summary | Tool | AIResourcesPlugin.get_resource | N/A |
| 101 | Phase 6 Delivery Account Summary | AI Assistant | DisplayPlugin.display_adaptive_card | 22,624 ms |
| 102 | Phase 6 Delivery Account Summary | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 104 | Phase 7 Client Summary | AI Assistant | AIResourcesPlugin.get_resource | 2,010 ms |
| 105 | Phase 7 Client Summary | Tool | AIResourcesPlugin.get_resource | N/A |
| 106 | Phase 7 Client Summary | AI Assistant | DisplayPlugin.display_adaptive_card | 8,151 ms |
| 107 | Phase 7 Client Summary | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 109 | Phase 8 Activation | AI Assistant | AIResourcesPlugin.get_resource | 1,795 ms |
| 110 | Phase 8 Activation | Tool | AIResourcesPlugin.get_resource | N/A |
| 111 | Phase 8 Activation | AI Assistant | LeadExecTools.update_client | 1,912 ms |
| 112 | Phase 8 Activation | Tool | LeadExecTools.update_client | 727 ms |
| 113 | Phase 8 Activation | AI Assistant | assistant_response: ✓ Setup complete. Your lead delivery system is now "ACTIVE" for BMQAP01. | 1,009 ms |

## Test 2

- Session ID: `69cc1a8c2697cf076414a9b3`
- Client Username: `bmqap02@example.com`

| Seq | Flow Phase | Row Type | Function / Label | Response Time |
|---:|---|---|---|---:|
| 5 | Flow Start | AI Assistant | SummaryPlugin.summarize_history | 10,432 ms |
| 6 | Flow Start | Tool | SummaryPlugin.summarize_history | 8 ms |
| 8 | Phase 1 Create Client | AI Assistant | AIResourcesPlugin.get_resource | 4,708 ms |
| 9 | Phase 1 Create Client | Tool | AIResourcesPlugin.get_resource | N/A |
| 10 | Phase 1 Create Client | AI Assistant | assistant_response: Great - first we'll set up your Client, Delivery Method, and Delivery Account. T... | 3,478 ms |
| 12 | Phase 1 Create Client | AI Assistant | LeadExecTools.create_client | 21,312 ms |
| 13 | Phase 1 Create Client | Tool | LeadExecTools.create_client | 608 ms |
| 14 | Phase 1 Create Client | AI Assistant | SummaryPlugin.summarize_history | 1,989 ms |
| 15 | Phase 1 Create Client | Tool | SummaryPlugin.summarize_history | 6 ms |
| 17 | Phase 2 Get Lead Types | AI Assistant | AIResourcesPlugin.get_resource | 5,443 ms |
| 18 | Phase 2 Get Lead Types | Tool | AIResourcesPlugin.get_resource | N/A |
| 19 | Phase 2 Get Lead Types | AI Assistant | LeadExecTools.get_lead_types | 999 ms |
| 20 | Phase 2 Get Lead Types | Tool | LeadExecTools.get_lead_types | 161 ms |
| 21 | Phase 2 Get Lead Types | AI Assistant | DisplayPlugin.display_adaptive_card | 24,835 ms |
| 22 | Phase 2 Get Lead Types | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 24 | Phase 2 Get Lead Types | AI Assistant | SummaryPlugin.summarize_history | 7,666 ms |
| 25 | Phase 2 Get Lead Types | Tool | SummaryPlugin.summarize_history | 7 ms |
| 27 | Phase 2 Get Lead Types | AI Assistant | AIResourcesPlugin.get_resource | 5,292 ms |
| 28 | Phase 2 Get Lead Types | Tool | AIResourcesPlugin.get_resource | N/A |
| 29 | Phase 2 Get Lead Types | AI Assistant | LeadExecTools.get_lead_types | 767 ms |
| 30 | Phase 2 Get Lead Types | Tool | LeadExecTools.get_lead_types | 268 ms |
| 31 | Phase 2 Get Lead Types | AI Assistant | DisplayPlugin.display_adaptive_card | 26,438 ms |
| 32 | Phase 2 Get Lead Types | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 34 | Phase 2 Get Lead Types | AI Assistant | SummaryPlugin.summarize_history | 8,314 ms |
| 35 | Phase 2 Get Lead Types | Tool | SummaryPlugin.summarize_history | 9 ms |
| 37 | Phase 2 Get Lead Types | AI Assistant | AIResourcesPlugin.get_resource | 5,901 ms |
| 38 | Phase 2 Get Lead Types | Tool | AIResourcesPlugin.get_resource | N/A |
| 39 | Phase 2 Get Lead Types | AI Assistant | LeadExecTools.get_lead_types | 9,982 ms |
| 40 | Phase 2 Get Lead Types | Tool | LeadExecTools.get_lead_types | 164 ms |
| 41 | Phase 2 Get Lead Types | AI Assistant | DisplayPlugin.display_adaptive_card | 26,521 ms |
| 42 | Phase 2 Get Lead Types | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 44 | Phase 2 Get Lead Types | AI Assistant | SummaryPlugin.summarize_history | 11,417 ms |
| 45 | Phase 2 Get Lead Types | Tool | SummaryPlugin.summarize_history | 18 ms |
| 47 | Phase 3 Create Delivery Method | AI Assistant | AIResourcesPlugin.get_resource | 7,661 ms |
| 48 | Phase 3 Create Delivery Method | Tool | AIResourcesPlugin.get_resource | N/A |
| 49 | Phase 3 Create Delivery Method | AI Assistant | DisplayPlugin.display_adaptive_card | 15,846 ms |
| 50 | Phase 3 Create Delivery Method | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 52 | Phase 3 Create Delivery Method | AI Assistant | DisplayPlugin.display_adaptive_card | 10,663 ms |
| 53 | Phase 3 Create Delivery Method | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 55 | Phase 3 Create Delivery Method | AI Assistant | assistant_response: What's your webhook URL where we should send the leads? | 7,722 ms |
| 57 | Phase 3 Create Delivery Method | AI Assistant | DisplayPlugin.display_adaptive_card | 14,046 ms |
| 58 | Phase 3 Create Delivery Method | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 60 | Phase 3 Create Delivery Method | AI Assistant | LeadExecTools.get_lead_type | 6,623 ms |
| 61 | Phase 3 Create Delivery Method | Tool | LeadExecTools.get_lead_type | 5,526 ms |
| 62 | Phase 3 Create Delivery Method | AI Assistant | DisplayPlugin.display_adaptive_card | 4,909 ms |
| 63 | Phase 3 Create Delivery Method | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 65 | Phase 3 Create Delivery Method | AI Assistant | assistant_response: Please paste the JSON schema that your client's API expects. | 4,357 ms |
| 67 | Phase 3 Create Delivery Method | AI Assistant | DisplayPlugin.display_adaptive_card | 26,489 ms |
| 68 | Phase 3 Create Delivery Method | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 70 | Phase 3 Create Delivery Method | AI Assistant | DisplayPlugin.display_adaptive_card | 55,505 ms |
| 71 | Phase 3 Create Delivery Method | Tool | DisplayPlugin.display_adaptive_card | 1 ms |
| 73 | Phase 3 Create Delivery Method | AI Assistant | LeadExecTools.create_delivery_method | 49,099 ms |
| 74 | Phase 3 Create Delivery Method | Tool | LeadExecTools.create_delivery_method | 754 ms |
| 75 | Phase 3 Create Delivery Method | AI Assistant | SummaryPlugin.summarize_history | 7,384 ms |
| 76 | Phase 3 Create Delivery Method | Tool | SummaryPlugin.summarize_history | 11 ms |
| 78 | Phase 3b Test Connection | AI Assistant | AIResourcesPlugin.get_resource | 5,417 ms |
| 79 | Phase 3b Test Connection | Tool | AIResourcesPlugin.get_resource | N/A |
| 80 | Phase 3b Test Connection | AI Assistant | DisplayPlugin.display_adaptive_card | 12,863 ms |
| 81 | Phase 3b Test Connection | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 83 | Phase 3b Test Connection | AI Assistant | SummaryPlugin.summarize_history | 8,284 ms |
| 84 | Phase 3b Test Connection | Tool | SummaryPlugin.summarize_history | 20 ms |
| 86 | Phase 4 Delivery Method Summary | AI Assistant | AIResourcesPlugin.get_resource | 5,837 ms |
| 87 | Phase 4 Delivery Method Summary | Tool | AIResourcesPlugin.get_resource | N/A |
| 88 | Phase 4 Delivery Method Summary | AI Assistant | DisplayPlugin.display_adaptive_card | 14,715 ms |
| 89 | Phase 4 Delivery Method Summary | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 91 | Phase 5 Create Delivery Account | AI Assistant | AIResourcesPlugin.get_resource | 3,707 ms |
| 92 | Phase 5 Create Delivery Account | Tool | AIResourcesPlugin.get_resource | N/A |
| 93 | Phase 5 Create Delivery Account | AI Assistant | assistant_response: Finally, let's set up your Delivery Account. Please provide the price per lead. | 10,708 ms |
| 95 | Phase 5 Create Delivery Account | AI Assistant | DisplayPlugin.display_adaptive_card | 16,124 ms |
| 96 | Phase 5 Create Delivery Account | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 98 | Phase 5 Create Delivery Account | AI Assistant | DisplayPlugin.display_adaptive_card | 2,159 ms |
| 99 | Phase 5 Create Delivery Account | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 101 | Phase 5 Create Delivery Account | AI Assistant | LeadExecTools.get_lead_type | 4,501 ms |
| 102 | Phase 5 Create Delivery Account | Tool | LeadExecTools.get_lead_type | 673 ms |
| 103 | Phase 5 Create Delivery Account | AI Assistant | assistant_response: Which states do you want to target? (e.g., CA, AZ, TX) | 13,856 ms |
| 105 | Phase 5 Create Delivery Account | AI Assistant | DisplayPlugin.display_adaptive_card | 29,641 ms |
| 106 | Phase 5 Create Delivery Account | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 108 | Phase 5 Create Delivery Account | AI Assistant | LeadExecTools.get_usa_states | 21,987 ms |
| 109 | Phase 5 Create Delivery Account | Tool | LeadExecTools.get_usa_states | 2,045 ms |
| 110 | Phase 5 Create Delivery Account | AI Assistant | LeadExecTools.create_delivery_account | 2,664 ms |
| 111 | Phase 5 Create Delivery Account | Tool | LeadExecTools.create_delivery_account | 657 ms |
| 112 | Phase 5 Create Delivery Account | AI Assistant | SummaryPlugin.summarize_history | 2,458 ms |
| 113 | Phase 5 Create Delivery Account | Tool | SummaryPlugin.summarize_history | 17 ms |
| 115 | Phase 6 Delivery Account Summary | AI Assistant | AIResourcesPlugin.get_resource | 8,233 ms |
| 116 | Phase 6 Delivery Account Summary | Tool | AIResourcesPlugin.get_resource | N/A |
| 117 | Phase 6 Delivery Account Summary | AI Assistant | DisplayPlugin.display_adaptive_card | 22,857 ms |
| 118 | Phase 6 Delivery Account Summary | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 120 | Phase 7 Client Summary | AI Assistant | AIResourcesPlugin.get_resource | 3,460 ms |
| 121 | Phase 7 Client Summary | Tool | AIResourcesPlugin.get_resource | N/A |
| 122 | Phase 7 Client Summary | AI Assistant | DisplayPlugin.display_adaptive_card | 8,539 ms |
| 123 | Phase 7 Client Summary | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 125 | Phase 8 Activation | AI Assistant | AIResourcesPlugin.get_resource | 2,787 ms |
| 126 | Phase 8 Activation | Tool | AIResourcesPlugin.get_resource | N/A |
| 127 | Phase 8 Activation | AI Assistant | LeadExecTools.update_client | 2,226 ms |
| 128 | Phase 8 Activation | Tool | LeadExecTools.update_client | 718 ms |
| 129 | Phase 8 Activation | AI Assistant | assistant_response: ✓ Setup complete. Your lead delivery system is now "ACTIVE" for BMQAP02. | 1,282 ms |

## Test 3

- Session ID: `69cc1d752697cf076414aa0f`
- Client Username: `bmqap03@example.com`

| Seq | Flow Phase | Row Type | Function / Label | Response Time |
|---:|---|---|---|---:|
| 5 | Flow Start | AI Assistant | SummaryPlugin.summarize_history | 14,749 ms |
| 6 | Flow Start | Tool | SummaryPlugin.summarize_history | 4 ms |
| 8 | Phase 1 Create Client | AI Assistant | AIResourcesPlugin.get_resource | 5,693 ms |
| 9 | Phase 1 Create Client | Tool | AIResourcesPlugin.get_resource | N/A |
| 10 | Phase 1 Create Client | AI Assistant | assistant_response: Great - first we'll set up your Client, Delivery Method, and Delivery Account. T... | 7,756 ms |
| 12 | Phase 1 Create Client | AI Assistant | LeadExecTools.create_client | 22,355 ms |
| 13 | Phase 1 Create Client | Tool | LeadExecTools.create_client | 4,066 ms |
| 14 | Phase 1 Create Client | AI Assistant | SummaryPlugin.summarize_history | 4,252 ms |
| 15 | Phase 1 Create Client | Tool | SummaryPlugin.summarize_history | 11 ms |
| 17 | Phase 2 Get Lead Types | AI Assistant | AIResourcesPlugin.get_resource | 5,863 ms |
| 18 | Phase 2 Get Lead Types | Tool | AIResourcesPlugin.get_resource | N/A |
| 19 | Phase 2 Get Lead Types | AI Assistant | LeadExecTools.get_lead_types | 973 ms |
| 20 | Phase 2 Get Lead Types | Tool | LeadExecTools.get_lead_types | 436 ms |
| 21 | Phase 2 Get Lead Types | AI Assistant | DisplayPlugin.display_adaptive_card | 25,931 ms |
| 22 | Phase 2 Get Lead Types | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 24 | Phase 2 Get Lead Types | AI Assistant | SummaryPlugin.summarize_history | 8,613 ms |
| 25 | Phase 2 Get Lead Types | Tool | SummaryPlugin.summarize_history | 7 ms |
| 27 | Phase 3 Create Delivery Method | AI Assistant | AIResourcesPlugin.get_resource | 5,597 ms |
| 28 | Phase 3 Create Delivery Method | Tool | AIResourcesPlugin.get_resource | N/A |
| 29 | Phase 3 Create Delivery Method | AI Assistant | DisplayPlugin.display_adaptive_card | 11,129 ms |
| 30 | Phase 3 Create Delivery Method | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 32 | Phase 3 Create Delivery Method | AI Assistant | DisplayPlugin.display_adaptive_card | 16,498 ms |
| 33 | Phase 3 Create Delivery Method | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 35 | Phase 3 Create Delivery Method | AI Assistant | assistant_response: What's your webhook URL where we should send the leads? | 4,702 ms |
| 37 | Phase 3 Create Delivery Method | AI Assistant | DisplayPlugin.display_adaptive_card | 7,225 ms |
| 38 | Phase 3 Create Delivery Method | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 40 | Phase 3 Create Delivery Method | AI Assistant | LeadExecTools.get_lead_type | 3,398 ms |
| 41 | Phase 3 Create Delivery Method | Tool | LeadExecTools.get_lead_type | 379 ms |
| 42 | Phase 3 Create Delivery Method | AI Assistant | DisplayPlugin.display_adaptive_card | 5,431 ms |
| 43 | Phase 3 Create Delivery Method | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 45 | Phase 3 Create Delivery Method | AI Assistant | assistant_response: Please paste the JSON schema that your client's API expects. | 5,087 ms |
| 47 | Phase 3 Create Delivery Method | AI Assistant | DisplayPlugin.display_adaptive_card | 45,782 ms |
| 48 | Phase 3 Create Delivery Method | Tool | DisplayPlugin.display_adaptive_card | 1 ms |
| 50 | Phase 3 Create Delivery Method | AI Assistant | LeadExecTools.create_delivery_method | 14,689 ms |
| 51 | Phase 3 Create Delivery Method | Tool | LeadExecTools.create_delivery_method | 2,556 ms |
| 52 | Phase 3 Create Delivery Method | AI Assistant | SummaryPlugin.summarize_history | 4,335 ms |
| 53 | Phase 3 Create Delivery Method | Tool | SummaryPlugin.summarize_history | 12 ms |
| 55 | Phase 3b Test Connection | AI Assistant | AIResourcesPlugin.get_resource | 6,057 ms |
| 56 | Phase 3b Test Connection | Tool | AIResourcesPlugin.get_resource | N/A |
| 57 | Phase 3b Test Connection | AI Assistant | DisplayPlugin.display_adaptive_card | 14,000 ms |
| 58 | Phase 3b Test Connection | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 60 | Phase 3b Test Connection | AI Assistant | SummaryPlugin.summarize_history | 6,663 ms |
| 61 | Phase 3b Test Connection | Tool | SummaryPlugin.summarize_history | 13 ms |
| 63 | Phase 4 Delivery Method Summary | AI Assistant | AIResourcesPlugin.get_resource | 6,962 ms |
| 64 | Phase 4 Delivery Method Summary | Tool | AIResourcesPlugin.get_resource | N/A |
| 65 | Phase 4 Delivery Method Summary | AI Assistant | DisplayPlugin.display_adaptive_card | 13,501 ms |
| 66 | Phase 4 Delivery Method Summary | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 68 | Phase 5 Create Delivery Account | AI Assistant | AIResourcesPlugin.get_resource | 4,878 ms |
| 69 | Phase 5 Create Delivery Account | Tool | AIResourcesPlugin.get_resource | N/A |
| 70 | Phase 5 Create Delivery Account | AI Assistant | assistant_response: Finally, let's set up your Delivery Account. Please provide the price per lead. | 11,154 ms |
| 72 | Phase 5 Create Delivery Account | AI Assistant | DisplayPlugin.display_adaptive_card | 22,682 ms |
| 73 | Phase 5 Create Delivery Account | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 75 | Phase 5 Create Delivery Account | AI Assistant | DisplayPlugin.display_adaptive_card | 12,462 ms |
| 76 | Phase 5 Create Delivery Account | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 78 | Phase 5 Create Delivery Account | AI Assistant | LeadExecTools.get_lead_type | 6,107 ms |
| 79 | Phase 5 Create Delivery Account | Tool | LeadExecTools.get_lead_type | 264 ms |
| 80 | Phase 5 Create Delivery Account | AI Assistant | assistant_response: Which states do you want to target? (e.g., CA, AZ, TX) | 31,140 ms |
| 82 | Phase 5 Create Delivery Account | AI Assistant | DisplayPlugin.display_adaptive_card | 45,454 ms |
| 83 | Phase 5 Create Delivery Account | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 85 | Phase 5 Create Delivery Account | AI Assistant | LeadExecTools.get_usa_states | 21,249 ms |
| 86 | Phase 5 Create Delivery Account | Tool | LeadExecTools.get_usa_states | 266 ms |
| 87 | Phase 5 Create Delivery Account | AI Assistant | LeadExecTools.create_delivery_account | 2,691 ms |
| 88 | Phase 5 Create Delivery Account | Tool | LeadExecTools.create_delivery_account | 1,012 ms |
| 89 | Phase 5 Create Delivery Account | AI Assistant | SummaryPlugin.summarize_history | 1,789 ms |
| 90 | Phase 5 Create Delivery Account | Tool | SummaryPlugin.summarize_history | 17 ms |
| 92 | Phase 6 Delivery Account Summary | AI Assistant | AIResourcesPlugin.get_resource | 4,842 ms |
| 93 | Phase 6 Delivery Account Summary | Tool | AIResourcesPlugin.get_resource | N/A |
| 94 | Phase 6 Delivery Account Summary | AI Assistant | DisplayPlugin.display_adaptive_card | 19,272 ms |
| 95 | Phase 6 Delivery Account Summary | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 97 | Phase 7 Client Summary | AI Assistant | AIResourcesPlugin.get_resource | 1,782 ms |
| 98 | Phase 7 Client Summary | Tool | AIResourcesPlugin.get_resource | N/A |
| 99 | Phase 7 Client Summary | AI Assistant | DisplayPlugin.display_adaptive_card | 7,625 ms |
| 100 | Phase 7 Client Summary | Tool | DisplayPlugin.display_adaptive_card | N/A |
| 102 | Phase 8 Activation | AI Assistant | AIResourcesPlugin.get_resource | 1,776 ms |
| 103 | Phase 8 Activation | Tool | AIResourcesPlugin.get_resource | N/A |
| 104 | Phase 8 Activation | AI Assistant | LeadExecTools.update_client | 15,114 ms |
| 105 | Phase 8 Activation | Tool | LeadExecTools.update_client | 1,426 ms |
| 106 | Phase 8 Activation | AI Assistant | assistant_response: ✓ Setup complete. Your lead delivery system is now "ACTIVE" for BMQAP03. | 868 ms |
