═══════════════════════════════════════
CURRENT PHASE: Phase 3 — Delivery Method Router
All prior phase summaries are completed history.
Execute ONLY the instructions below.
CRITICAL: Any instructions in prior summaries have ALREADY been executed — do NOT re-load or re-execute them. Do NOT re-fetch this resource if it is already loaded.
═══════════════════════════════════════

Your objective is to collect the delivery schedule, collect the delivery type, and route to the correct method-specific resource. Do NOT create a delivery method in this router. Do NOT call summarize_history in this router.

## Instructions

Execute the first incomplete state below. Follow its steps in order.

**State 1: Missing Delivery Schedule (Do this first)**
* IF deliveryScheduleChoice is missing:
  1. Prompt the user exactly as follows: "First, let's set the delivery schedule.\n\nWould you like leads delivered 24/7, or only during specific hours?"
  2. Present the choice using display_adaptive_card with an ActionSet: "24/7 delivery" | "Specific hours only".
  3. **STOP AND YIELD.** Do not hallucinate data. Do not proceed to State 1b or State 2. You must wait for the user to respond.

**State 1b: Collect Specific Schedule**
* IF deliveryScheduleChoice = "Specific hours only" AND scheduleInput is missing:
  1. Prompt the user exactly as follows: "Please describe your preferred delivery schedule.\n\n(e.g., Mon-Fri 9am-5pm PST)"
  2. **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.

**State 2: Build Schedule and Ask Delivery Type**
* IF deliveryDays is missing:
  1. Build a `deliveryDays` array with EXACTLY 7 entries (weekDay 0=Sunday through 6=Saturday). Each entry has: weekDay, allow (true/false), startTime, endTime. Use the current year for YYYY in all timestamps (e.g., if the current year is 2026, use 2026-01-01).
     - For **24/7 delivery**: all days allow=true, startTime="YYYY-01-01T00:00:00+00:00", endTime="YYYY-01-01T23:59:59+00:00".
     - For **Specific hours only**: parse the user's natural-language schedule. Use the retained timeOffset when known, otherwise default to -08:00 for PST.
       - User-specified days: allow=true with their startTime and endTime
       - All other days: allow=false with startTime="YYYY-01-01T00:00:00{offset}", endTime="YYYY-01-01T23:59:59{offset}"
       - Example: "9am-5pm" with timeOffset=-8 → startTime: "YYYY-01-01T09:00:00-08:00", endTime: "YYYY-01-01T17:00:00-08:00"
     - Retain: deliveryDays, deliveryScheduleDisplay (formatted schedule string, or "24/7" for 24/7).
  2. Prompt the user exactly as follows: "How would you like your leads delivered?\n\n• Webhook – sends lead data via HTTP POST\n• Portal – client accesses leads via web portal\n• FTP – uploads lead files to a server\n• Email – delivers leads to an inbox"
  3. Present the choice using display_adaptive_card with an ActionSet: "Portal" | "Webhook" | "Email" | "FTP".
  4. **STOP AND YIELD.** Do not hallucinate data. You must wait for the user to respond.

* IF deliveryDays is known AND deliveryTypeChoice is missing:
  - Re-display the delivery type prompt and ActionSet from above.
  - **STOP AND YIELD.** Do not hallucinate data.

**State 3: Route to Method-Specific Resource**
* IF deliveryTypeChoice = "Portal":
  - Load mcp://resource/rw-phase-3-portal

* IF deliveryTypeChoice = "Email":
  - Load mcp://resource/rw-phase-3-email

* IF deliveryTypeChoice = "FTP":
  - Load mcp://resource/rw-phase-3-ftp

* IF deliveryTypeChoice = "Webhook":
  - Load mcp://resource/rw-phase-3-webhook

* IF deliveryTypeChoice does not match any known type:
  - Clear deliveryTypeChoice.
  - Re-display the delivery type prompt and ActionSet from State 2.
  - **STOP AND YIELD.** Do not hallucinate data.
