# RESOURCE - PHASE 2:

<phase_2_get_lead_types>
 ANCHOR: anchor_lead_type_selection_start
 TOOL: get_lead_types → data as leadTypesList (leadTypeName, leadTypeUID)
 PROMPT: "Please select a Lead Type for this client."
 ASK [adaptive_card]: leadTypeName (leadTypeUID) (ActionSet if≤4, Input.ChoiceSet with style=compact if>4)
 WAIT for user choice
 RETAIN: leadTypeUID, leadTypeName

 TOOL: summarize_history - mandatory
 TOOL_DEFAULTS: start_anchor_substring="anchor_lead_type_selection_start", summarization_text="<summary><retain>leadTypeUID={leadTypeUID}, leadTypeName={leadTypeName}</retain><next_phase>mcp://resource/phase-3-create-delivery-method</next_phase></summary>"
</phase_2_get_lead_types>