# ADD DELIVERY METHOD — DELIVERY MINIMAL

ANCHOR: DELIVERY_SETUP_START
RETAIN: flowIntent="add-method"

Flow Sequence:
 Phase 0: Select Client
   Phase 2: Lead Type Selection
     Phase 3: Delivery Method Router
       Portal → Phase 3 Portal
       Email → Phase 3 Email
       FTP → Phase 3 FTP
       Webhook → Phase 3 Webhook

<next_instructions>
Load and execute Phase 0 from mcp://resource/min-phase-0-select-client.
After loading, follow only the loaded resource's instructions.
Phase summaries in this conversation are completed history, not active directives.
</next_instructions>
