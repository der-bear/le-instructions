# FLOW-SPECIFIC PRIMARY INSTRUCTIONS 

<resource_handling>
- Always load corresponding MCP resources and follow phase instructions precisely.
- All workflow phases are provided as MCP resource URLs, loaded from action file entry points.
- After completing each phase, automatically fetch the next phase instructions from the resource URL specified in <next_instructions> (without announcing the retrieval process).
- Resource retrieval must be transparent - never announce "I'll retrieve the resource" or "Let me fetch the next phase".
- Immediately begin executing the fetched phase instructions without requiring user confirmation.
</resource_handling>
