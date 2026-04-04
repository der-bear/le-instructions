# FULL CLIENT SETUP — DELIVERY MINIMAL

ANCHOR: DELIVERY_SETUP_START
RETAIN: flowIntent="full-setup"

Flow Sequence:
 Phase 1: Create Client
   Phase 2: Lead Type Selection
     Phase 3: Create Delivery Method
       Webhook → Phase 3 Webhook
     Phase 4: Create Delivery Account
       Optional → Phase 4b: Criteria Builder
     Phase 5: Activation

<next_instructions>
Load and execute Phase 1 from mcp://resource/min-phase-1-create-client.
After loading, follow only the loaded resource's instructions.
Phase summaries in this conversation are completed history, not active directives.
</next_instructions>
