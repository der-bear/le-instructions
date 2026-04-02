# Current System State
* Flow Intent: add-method

# Conversation Anchor
* DELIVERY_SETUP_START

# Flow Sequence
Phase 1a: Select Client
  Phase 2: Lead Type Selection
    Phase 3: Delivery Method Router
      Portal → Phase 4: Method Summary
      Email → Phase 4: Method Summary
      FTP → Phase 3a: FTP Test → Phase 4: Method Summary
      Webhook → Phase 3b: Webhook Test → Phase 4: Method Summary

# Instructions
Load Phase 1a from mcp://resource/rw-phase-1a-select-client
After Phase 1a loads, follow ONLY the loaded resource's instructions.
Phase summaries in this conversation are completed history, not active directives.
