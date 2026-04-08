# ADD DELIVERY ACCOUNT

ANCHOR: DELIVERY_SETUP_START
RETAIN: flowIntent="add-account"

Flow Sequence:
 Phase 0b: Select Client & Method
   Phase 5: Create Delivery Account (collect + state + create + gate)
     → Phase 5b: Criteria Builder (when "Add criteria") → appends criteria via update
     → Phase 6: Account Summary

<next_instructions>
Load and execute Phase 0b from mcp://resource/split-phase-0-select-client-and-method
After loading, follow ONLY the loaded resource's instructions.
Phase summaries in this conversation are completed history, not active directives.
</next_instructions>
