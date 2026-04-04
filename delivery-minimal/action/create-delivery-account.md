# ADD DELIVERY ACCOUNT — DELIVERY MINIMAL

ANCHOR: DELIVERY_SETUP_START
RETAIN: flowIntent="add-account"

Flow Sequence:
 Phase 0: Select Client And Method
   Phase 4: Create Delivery Account
     Optional → Phase 4b: Criteria Builder

<next_instructions>
Load and execute Phase 0 from mcp://resource/min-phase-0-select-client-and-method.
After loading, follow only the loaded resource's instructions.
Phase summaries in this conversation are completed history, not active directives.
</next_instructions>
