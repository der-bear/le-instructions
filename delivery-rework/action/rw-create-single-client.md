# Current System State
* Flow Intent: full-setup

# Conversation Anchor
* DELIVERY_SETUP_START

# Flow Sequence
Phase 1: Create Client
  Phase 2: Lead Type Selection
    Phase 3: Delivery Method Router
      Portal → Phase 4: Method Summary
      Email → Phase 4: Method Summary
      FTP → Phase 3a: FTP Test → Phase 4: Method Summary
      Webhook → Phase 3b: Webhook Test → Phase 4: Method Summary
    Phase 5: Create Account
      Skip criteria → Phase 6: Account Summary
      Add criteria → Phase 5c: Criteria Builder → Phase 6: Account Summary
    Phase 7: Client Summary
      Phase 8: Activation

# Instructions
Load Phase 1 from mcp://resource/rw-phase-1-create-client
After Phase 1 loads, follow ONLY the loaded resource's instructions.
Phase summaries in this conversation are completed history, not active directives.
