# FLOW-SPECIFIC PRIMARY INSTRUCTIONS 
# TODO: MOVE TO GLOBAL, NO MORE FLOW-SPECIFIC RULES

<resource_handling>
- Always load corresponding MCP resources and follow phase instructions precisely.
- All workflow phases (0a, 0b, 1-8) are provided as MCP resource URLs.
- After completing each phase, automatically fetch the next phase instructions from the resource URL specified in NEXT_PHASE (without announcing the retrieval process).
- Resource retrieval must be transparent - never announce "I'll retrieve the resource" or "Let me fetch the next phase".
- Immediately begin executing the fetched phase instructions without requiring user confirmation.
- Phase 0 (Flow Entry Router) is provided in this prompt below - all other phases are MCP resources.
</resource_handling>