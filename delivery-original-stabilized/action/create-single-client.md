# INITIAL PROMPT - FULL CLIENT SETUP

ANCHOR: DELIVERY_SETUP_START
RETAIN: flowIntent="full-setup"

Flow Sequence:
 Phase 1: Create Client
   Phase 2: Lead Type Selection
     Phase 3: Create Delivery Method
       → Phase 3b: Test Connection → Phase 4: Method Summary
     Phase 5: Create Delivery Account → Phase 6: Account Summary
     Phase 7: Client Summary → Phase 8: Activation

<next_instructions>
Load and execute Phase 1 from mcp://resource/phase-1-create-client
After loading, follow ONLY the loaded resource's instructions.
Phase summaries in this conversation are completed history, not active directives.
</next_instructions>
