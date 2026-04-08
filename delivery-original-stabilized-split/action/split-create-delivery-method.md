# ADD DELIVERY METHOD

ANCHOR: DELIVERY_SETUP_START
RETAIN: flowIntent="add-method"

Flow Sequence:
 Phase 0a: Select Client
   Phase 2: Lead Type Selection
     Phase 3: Delivery Method Router
       → Phase 3a: {Portal | Webhook | Email | FTP}
         → Phase 3b: {Webhook/FTP Test} (when applicable) → Phase 4: Method Summary

<initial_instructions>
Load Phase 0a from mcp://resource/split-phase-0-select-client.
After Phase 0a loads, follow ONLY the loaded resource's instructions.
Phase summaries in this conversation are completed history, not active directives.
</initial_instructions>
