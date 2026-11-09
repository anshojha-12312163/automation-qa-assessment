# Automation & QA Developer Assessment

Submission by **<Ansh ojha>**.

- Loom walkthrough: <add-your-loom-link>

## Contents

| File | What it is |
|---|---|
| `Task1_QA_Report_<YourName>.docx` | QA report on the RealWorld (Conduit) demo app: bug table and root-cause analysis |
| `Task2_Workflow_<YourName>.json` | n8n workflow: HackerNews API digest to Discord |
| `Bonus_UptimeMonitor_<YourName>.json` | n8n uptime monitor for the app tested in Task 1 |
| `screenshots/` | Workflow canvas and successful execution screenshots |

## Task 1: QA report
App tested: https://demo.realworld.io. The report lists the issues found in the bug template format (title, steps, expected vs actual, severity, suspected cause) and a root-cause analysis of one issue: the auth token stored in localStorage, why it is risky, and how to fix it (HttpOnly cookie, CSRF protection, short-lived tokens, CSP).

## Task 2: Morning brief workflow
**Flow:** Schedule (hourly) → HN Algolia front page → Code node keeps top 5 → second endpoint enriches each story → digest built → IF (any story over 300 points?) → Discord.

- **APIs:** HN Algolia Search and Items endpoints. Free, no API key, stable JSON.
- **Transformation:** filter out items without titles, sort by points, keep 5, reshape fields. Stories without an external URL link to the HN discussion.
- **IF threshold (300 points):** top front-page stories usually score 100 to 500, so 300 marks a busy day. Hot days get a different header.
- **Error handling:**
  - The main fetch retries 3 times, then routes to a Discord alert.
  - The enrichment call continues on error, so one failed item does not stop the run.
  - A separate Error Trigger branch reports any other crash (set this workflow as its own error workflow in Workflow Settings).
- **Credentials:** no secrets in the JSON. Nodes use an n8n credential named "Discord Webhook".

## Bonus: uptime monitor
- Pings the app every 5 minutes with 3 retries and a 10 second timeout.
- Records response time and status in workflow static data.
- Any status other than 200 sends a Discord alert.
- A daily 9 AM summary reports total checks, failures, uptime percentage, and average and slowest response time, then resets the counters.

## How to run
1. In n8n, choose Import from File and select a workflow JSON.
2. Create a **Discord Webhook** credential (paste your webhook URL) and attach it to every Discord node.
3. Click Execute workflow and check your Discord channel.
4. For the monitor, activate the workflow so the schedules run.

## Notes and limitations
- Static data in the monitor only persists while the workflow is active (not on manual test runs).
- The uptime monitor targets the public demo; replace the URL in the "Ping App" node to monitor another app.
