# Current System State
* Flow Intent: add-account

# Conversation Anchor
* DELIVERY_SETUP_START

# Flow Sequence
Phase 1b: Select Client & Method
  Phase 5: Create Account
    Skip criteria → Phase 6: Account Summary
    Add criteria → Phase 5c: Criteria Builder → Phase 6: Account Summary

# Instructions
Load Phase 1b from mcp://resource/rw-phase-1b-select-client-and-method
After Phase 1b loads, follow ONLY the loaded resource's instructions.
Phase summaries in this conversation are completed history, not active directives.
