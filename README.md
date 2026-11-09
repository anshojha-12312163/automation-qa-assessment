# Task 2 README: Morning Brief (HackerNews Digest)

## What it does
Every hour (Schedule trigger, for testing) the workflow fetches the HackerNews front page, keeps the top 5 stories by points, enriches each with its discussion details, and posts a digest to a Discord channel.

## APIs used and why
- **HN Algolia Search API** (`hn.algolia.com/api/v1/search?tags=front_page`): free, no API key, stable JSON, fast. Good for a reliable demo.
- **HN Algolia Items API** (`/api/v1/items/{id}`): second endpoint of the same API. Used to enrich each story with its top-level reply count.

## Transformation (Code node "Keep Top 5")
Filters out hits without a title, sorts by points descending, keeps 5, and reshapes each item to `{objectID, title, url, points, comments, author}`. Stories with no external URL fall back to the HN discussion link.

## Conditional branch
IF node "Any Story Over 300 Points?" checks the highest score in the digest.
- **True**: sends the digest with a "Hot morning brief" header.
- **False**: sends a normal "Morning brief".

The 300 threshold was chosen because front-page top stories usually sit in the 100-500 range, so 300 marks an unusually busy day. It is easy to change in the IF node.

## Error handling (nothing fails silently)
1. **Fetch Front Page**: retries 3 times (2s apart); if it still fails, the error output goes to "Alert Fetch Failed" (a Discord message).
2. **Enrich With Item Details**: set to "Continue (regular output)", so one failed item does not stop the run. The digest shows "enrichment unavailable" for that story.
3. **Workflow Error Trigger** (separate branch): any other crash posts the workflow name, failing node and message to Discord. Note: in n8n this trigger fires when this workflow is set as its own error workflow (Workflow Settings > Error workflow).

## Credentials
No secrets in the JSON. The Discord nodes reference a credential named "Discord Webhook" from the n8n Credentials store. After importing, open any Discord node, pick or create that credential (paste your webhook URL there), and relink the other Discord nodes to it.

## How to run
1. Import `Task2_Workflow_[YourName].json` in n8n.
2. Create the Discord Webhook credential and attach it to the 4 Discord nodes.
3. Click "Execute workflow" and check Discord.
