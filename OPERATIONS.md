# Operations

## Deploying a workflow version

1. Import the new workflow JSON into n8n.
2. Activate it.
3. Deactivate and archive every older copy — multiple active copies compete for the same webhooks and the MCP endpoint, and calls may silently hit the old version.
4. In the ElevenLabs agent settings, re-sync the MCP tool list (tool schemas are cached and do not refresh on their own).
5. Update production/ in this repository with the deployed artifacts.

## Deploying a prompt version

1. Paste the new prompt into the ElevenLabs agent.
2. Verify dynamic variables used by the prompt (business_hours, caller_phone) have defaults set in the agent's dynamic-variable settings.
3. Update production/ in this repository.

## Debugging a bad call

1. n8n → Executions → open the execution matching the call time.
2. Click the suspect node: the INPUT panel shows exactly what arrived; the OUTPUT panel shows what it returned. Most historical incidents were resolved by comparing the two.
3. For estimate issues: check the trigger data for the service field, then the estimate node's output message (MISSING INPUT and sheet-unavailable states name themselves).

## Known hazards

- Google OAuth apps in Testing mode expire refresh tokens every 7 days; publish the OAuth consent screen to Production. Sheets nodes are configured to continue on error, so a dead credential degrades silently.
- Google Sheets CSV export (gviz) can serve stale data; trust the n8n node read over the export URL.
- Status strings (Booked, Rescheduled, Cancelled, Rebooked) are matched by workflow code; renaming them in the sheet breaks reminders, cancellation, and rescheduling.
- Business configuration exists as three synchronized n8n Set nodes (main / Init / Reminder); update all three.

## Rollback

Import the previous workflow JSON from production/ history, activate it, deactivate the newer one, and re-sync the ElevenLabs tool list.
