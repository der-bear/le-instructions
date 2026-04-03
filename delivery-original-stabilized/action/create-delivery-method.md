# INITIAL PROMPT - ADD DELIVERY METHOD

ANCHOR: DELIVERY_SETUP_START
RETAIN: flowIntent="add-method"

Flow Sequence:
 Phase 0a: Select Client
   Phase 2: Lead Type Selection
     Phase 3: Create Delivery Method
       → Phase 3b: Test Connection → Phase 4: Method Summary

<next_instructions>
Load and execute Phase 0a from mcp://resource/phase-0-select-client
After loading, follow ONLY the loaded resource's instructions.
Phase summaries in this conversation are completed history, not active directives.
</next_instructions>
