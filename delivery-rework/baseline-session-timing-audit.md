# Baseline Session Timing Audit

## Scope

This audit reviews the 3 completed baseline sessions referenced by [baseline-test-protocol.md](/Users/aderkach/Source/le-instructions/delivery/baseline-test-protocol.md).

Measured artifacts:

- `/Users/aderkach/Downloads/69cc18912697cf076414a968.html`
- `/Users/aderkach/Downloads/69cc1a8c2697cf076414a9b3.html`
- `/Users/aderkach/Downloads/69cc1d752697cf076414aa0f.html`

What was measured:

- every assistant message with a visible `Response time: ...ms` badge
- every tool message with a visible `Response time: ...ms` badge
- every tool message without a timing badge is still listed, but marked `N/A`

What the totals mean:

- `Assistant Total ms` = sum of all timed assistant responses in the saved HTML
- `Tool Total ms` = sum of all timed tool messages in the saved HTML
- `Observed Timed Total ms` = `Assistant Total ms + Tool Total ms`

What is not included:

- user think time
- any timing not exposed in the saved HTML
- any tool message that has no response-time badge

## Accuracy Cross-Check

The timing extraction was verified in two independent passes:

1. DOM parsing with BeautifulSoup over `div.message-item`
2. regex-based parsing over the raw saved HTML

The two passes matched exactly on:

- assistant message counts
- tool message counts
- timed assistant totals
- timed tool totals

Agent spawning was not reliably available in this environment, so the cross-check was performed with two independent extraction methods instead.

## Summary

| Test ID | Client Username | Session ID | Assistant Timed Count | Assistant Total ms | Tool Timed Count | Tool Total ms | Observed Timed Total ms | Full Flow Completion | Final Action | Notable Variance |
|---|---|---|---|---|---|---|---|---|---|---|
| BL-01 | bmqap01@example.com | 69cc18912697cf076414a968 | 45 | 402776 | 17 | 17658 | 420434 | Yes | Activate | mapping 18 out of 22 fields; create_client error recovered |
| BL-02 | bmqap02@example.com | 69cc1a8c2697cf076414a9b3 | 51 | 590103 | 19 | 11671 | 601774 | Yes | Activate | mapping 17 out of 22 fields |
| BL-03 | bmqap03@example.com | 69cc1d752697cf076414aa0f | 42 | 478126 | 15 | 10470 | 488596 | Yes | Activate | mapping 15 out of 22 fields |

## Key Findings

- All 3 sessions completed the full flow successfully.
- All 3 sessions deviated from the current protocol by ending with `Activate` instead of `Keep Inactive`.
- Assistant-side timed processing dominates the observed total in every run:
  - BL-01: 95.80%
  - BL-02: 98.06%
  - BL-03: 97.86%
- Mapping results were not stable across the 3 runs even though the input payload was intended to be the same:
  - BL-01: 18 of 22 fields mapped
  - BL-02: 17 of 22 fields mapped
  - BL-03: 15 of 22 fields mapped
- BL-01 shows one recoverable `create_client` deserialization error before succeeding.
- The protocol says `Landing Tree`, but the saved session outputs consistently show `LendingTree`.

## BL-01

Session ID: `69cc18912697cf076414a968`  
Client Username: `bmqap01@example.com`  
Observed Timed Total: `420434ms`  
Full Flow Completion: `Yes`  
Final Action: `Activate`

### Assistant Responses

```text
seq  ms     label
5    8452   summarize_history
8    4373   get_resource
10   6511   Great - first we'll set up your Client, Delivery Method, and Delivery
12   25390  create_client
14   13712  create_client
16   8006   summarize_history
19   3828   get_resource
21   1128   get_lead_types
23   17125  display_adaptive_card
26   7116   summarize_history
29   5026   get_resource
31   7914   display_adaptive_card
34   10896  display_adaptive_card
37   4880   What's your webhook URL where we should send the leads?
39   13528  display_adaptive_card
42   5232   get_lead_type
44   3422   display_adaptive_card
47   1631   get_resource
49   4774   Please paste the JSON schema that your client's API expects.
51   51790  display_adaptive_card
54   14388  create_delivery_method
56   5414   summarize_history
59   3673   get_resource
61   9127   display_adaptive_card
64   6777   summarize_history
67   6216   get_resource
69   12649  display_adaptive_card
72   3050   get_resource
74   15719  Finally, let's set up your Delivery Account. Please provide the price
76   17454  display_adaptive_card
79   2851   display_adaptive_card
82   3697   get_lead_type
84   9479   Which states do you want to target? (e.g., CA, AZ, TX)
86   27963  display_adaptive_card
89   8857   get_usa_states
91   2296   display_adaptive_card
94   2695   create_delivery_account
96   1973   summarize_history
99   6263   get_resource
101  22624  display_adaptive_card
104  2010   get_resource
106  8151   display_adaptive_card
109  1795   get_resource
111  1912   update_client
113  1009   ✓ Setup complete. Your lead delivery system is now "ACTIVE" for BMQAP0
```

### Tool Messages

```text
seq  ms    label
6    11    summary_accepted
9    N/A   phase_1_create_client
13   4687  An error occurred invoking 'create_client': JSON deserialization for t
15   2417  tool_result_json
17   8     summary_accepted
20   N/A   phase_2_get_lead_types
22   258   tool_result_json
24   N/A   Please select a Lead Type for this client. Select a Lead Type Dup Chec
27   9     summary_accepted
30   N/A   phase_3_create_delivery_method
32   N/A   First, let's set the delivery schedule. Would you like leads delivered
35   2     How would you like your leads delivered? • Webhook – sends lead data v
40   N/A   Would you like to configure field mappings? If you have posting instru
43   2866  tool_result_json
45   N/A   What content type should this delivery use? URL Encoded JSON XML I'm n
48   N/A   phase_3_create_delivery_method
52   1     Field Mapping Preview System Field Delivery Field Status FirstName fir
55   2396  tool_result_json
57   13    summary_accepted
60   N/A   phase_3b_test_connection
62   N/A   Would you like to test the connection to your endpoint before continui
65   12    summary_accepted
68   N/A   phase_4_delivery_method_summary
70   N/A   Delivery Method Created Company Name BMQAP01 Lead Type LendingTree Del
73   N/A   phase_5_create_delivery_account
77   N/A   Will this client receive exclusive or shared leads? Exclusive Shared
80   N/A   Would you like to enable the Order System for this client? Yes No
83   2193  tool_result_json
87   N/A   Based on your LendingTree lead type, here are the most common criteria
90   453   tool_result_json
92   N/A   Please confirm the states you want to target: • CA • TX Confirm Edit
95   1588  tool_result_json
97   17    summary_accepted
100  N/A   phase_6_delivery_account_summary
102  N/A   Delivery Account Created Company Name BMQAP01 Lead Type LendingTree De
105  N/A   phase_7_client_summary
107  N/A   Client Setup Summary Company Name BMQAP01 Contact Email bmqap01@exampl
110  N/A   phase_8_activation
112  727   tool_result_json
```

## BL-02

Session ID: `69cc1a8c2697cf076414a9b3`  
Client Username: `bmqap02@example.com`  
Observed Timed Total: `601774ms`  
Full Flow Completion: `Yes`  
Final Action: `Activate`

### Assistant Responses

```text
seq  ms     label
5    10432  summarize_history
8    4708   get_resource
10   3478   Great - first we'll set up your Client, Delivery Method, and Delivery
12   21312  create_client
14   1989   summarize_history
17   5443   get_resource
19   999    get_lead_types
21   24835  display_adaptive_card
24   7666   summarize_history
27   5292   get_resource
29   767    get_lead_types
31   26438  display_adaptive_card
34   8314   summarize_history
37   5901   get_resource
39   9982   get_lead_types
41   26521  display_adaptive_card
44   11417  summarize_history
47   7661   get_resource
49   15846  display_adaptive_card
52   10663  display_adaptive_card
55   7722   What's your webhook URL where we should send the leads?
57   14046  display_adaptive_card
60   6623   get_lead_type
62   4909   display_adaptive_card
65   4357   Please paste the JSON schema that your client's API expects.
67   26489  display_adaptive_card
70   55505  display_adaptive_card
73   49099  create_delivery_method
75   7384   summarize_history
78   5417   get_resource
80   12863  display_adaptive_card
83   8284   summarize_history
86   5837   get_resource
88   14715  display_adaptive_card
91   3707   get_resource
93   10708  Finally, let's set up your Delivery Account. Please provide the price
95   16124  display_adaptive_card
98   2159   display_adaptive_card
101  4501   get_lead_type
103  13856  Which states do you want to target? (e.g., CA, AZ, TX)
105  29641  display_adaptive_card
108  21987  get_usa_states
110  2664   create_delivery_account
112  2458   summarize_history
115  8233   get_resource
117  22857  display_adaptive_card
120  3460   get_resource
122  8539   display_adaptive_card
125  2787   get_resource
127  2226   update_client
129  1282   ✓ Setup complete. Your lead delivery system is now "ACTIVE" for BMQAP0
```

### Tool Messages

```text
seq  ms    label
6    8     summary_accepted
9    N/A   phase_1_create_client
13   608   tool_result_json
15   6     summary_accepted
18   N/A   phase_2_get_lead_types
20   161   tool_result_json
22   N/A   Please select a Lead Type for this client. Select a lead type... Dup C
25   7     summary_accepted
28   N/A   phase_2_get_lead_types
30   268   tool_result_json
32   N/A   Please select a Lead Type for this client. Select a lead type Dup Chec
35   9     summary_accepted
38   N/A   phase_2_get_lead_types
40   164   tool_result_json
42   N/A   Please select a Lead Type for this client. Choose a Lead Type Dup Chec
45   18    summary_accepted
48   N/A   phase_3_create_delivery_method
50   N/A   First, let's set the delivery schedule. Would you like leads delivered
53   N/A   How would you like your leads delivered? • Webhook – sends lead data v
58   N/A   Would you like to configure field mappings? If you have posting instru
61   5526  tool_result_json
63   N/A   What content type should this delivery use? URL Encoded JSON XML I'm n
68   N/A   Which system field should we map the delivery field 'mobile_phone' to?
71   1     Field Mapping Preview System Field Delivery Field Status FirstName fir
74   754   tool_result_json
76   11    summary_accepted
79   N/A   phase_3b_test_connection
81   N/A   Would you like to test the connection to your endpoint before continui
84   20    summary_accepted
87   N/A   phase_4_delivery_method_summary
89   N/A   Delivery Method Created Company Name BMQAP02 Lead Type LendingTree Del
92   N/A   phase_5_create_delivery_account
96   N/A   Will this client receive exclusive or shared leads? Exclusive Shared
99   N/A   Would you like to enable the Order System for this client? Yes No
102  673   tool_result_json
106  N/A   Based on your LendingTree lead type, here are the most common criteria
109  2045  tool_result_json
111  657   tool_result_json
113  17    summary_accepted
116  N/A   phase_6_delivery_account_summary
118  N/A   Delivery Account Created Company Name BMQAP02 Lead Type LendingTree De
121  N/A   phase_7_client_summary
123  N/A   Client Setup Summary Company Name BMQAP02 Contact Email bmqap02@exampl
126  N/A   phase_8_activation
128  718   tool_result_json
```

## BL-03

Session ID: `69cc1d752697cf076414aa0f`  
Client Username: `bmqap03@example.com`  
Observed Timed Total: `488596ms`  
Full Flow Completion: `Yes`  
Final Action: `Activate`

### Assistant Responses

```text
seq  ms     label
5    14749  summarize_history
8    5693   get_resource
10   7756   Great - first we'll set up your Client, Delivery Method, and Delivery
12   22355  create_client
14   4252   summarize_history
17   5863   get_resource
19   973    get_lead_types
21   25931  display_adaptive_card
24   8613   summarize_history
27   5597   get_resource
29   11129  display_adaptive_card
32   16498  display_adaptive_card
35   4702   What's your webhook URL where we should send the leads?
37   7225   display_adaptive_card
40   3398   get_lead_type
42   5431   display_adaptive_card
45   5087   Please paste the JSON schema that your client's API expects.
47   45782  display_adaptive_card
50   14689  create_delivery_method
52   4335   summarize_history
55   6057   get_resource
57   14000  display_adaptive_card
60   6663   summarize_history
63   6962   get_resource
65   13501  display_adaptive_card
68   4878   get_resource
70   11154  Finally, let's set up your Delivery Account. Please provide the price
72   22682  display_adaptive_card
75   12462  display_adaptive_card
78   6107   get_lead_type
80   31140  Which states do you want to target? (e.g., CA, AZ, TX)
82   45454  display_adaptive_card
85   21249  get_usa_states
87   2691   create_delivery_account
89   1789   summarize_history
92   4842   get_resource
94   19272  display_adaptive_card
97   1782   get_resource
99   7625   display_adaptive_card
102  1776   get_resource
104  15114  update_client
106  868    ✓ Setup complete. Your lead delivery system is now "ACTIVE" for BMQAP0
```

### Tool Messages

```text
seq  ms    label
6    4     summary_accepted
9    N/A   phase_1_create_client
13   4066  tool_result_json
15   11    summary_accepted
18   N/A   phase_2_get_lead_types
20   436   tool_result_json
22   N/A   Please select a Lead Type for this client. Choose a lead type Dup Chec
25   7     summary_accepted
28   N/A   phase_3_create_delivery_method
30   N/A   First, let's set the delivery schedule. Would you like leads delivered
33   N/A   How would you like your leads delivered? • Webhook – sends lead data v
38   N/A   Would you like to configure field mappings? If you have posting instru
41   379   tool_result_json
43   N/A   What content type should this delivery use? URL Encoded JSON XML I'm n
48   1     Field Mapping Preview System Field Delivery Field Status FirstName fir
51   2556  tool_result_json
53   12    summary_accepted
56   N/A   phase_3b_test_connection
58   N/A   Would you like to test the connection to your endpoint before continui
61   13    summary_accepted
64   N/A   phase_4_delivery_method_summary
66   N/A   Delivery Method Created Company Name BMQAP03 Lead Type LendingTree Del
69   N/A   phase_5_create_delivery_account
73   N/A   Will this client receive exclusive or shared leads? Exclusive Shared
76   N/A   Would you like to enable the Order System for this client? Yes No
79   264   tool_result_json
83   N/A   Based on your LendingTree lead type, here are the most common criteria
86   266   tool_result_json
88   1012  tool_result_json
90   17    summary_accepted
93   N/A   phase_6_delivery_account_summary
95   N/A   Delivery Account Created Company Name BMQAP03 Lead Type LendingTree De
98   N/A   phase_7_client_summary
100  N/A   Client Setup Summary Company Name BMQAP03 Contact Email bmqap03@exampl
103  N/A   phase_8_activation
105  1426  tool_result_json
```
