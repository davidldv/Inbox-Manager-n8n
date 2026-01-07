# inbox-manager (n8n workflows)

This repo contains exported n8n workflows from [workflows.json](workflows.json).

## Inbox Manager (primary)

**Goal:** Automatically categorize Gmail messages and (for priority emails) generate a draft reply.

### What it does

This workflow has two paths:

1) **Auto-categorize new incoming emails** (runs continuously)
- Trigger: **Gmail Trigger** (polls every minute)
- Classifier: **Text Classifier** (LangChain) using **Google Gemini Chat Model**
- Actions based on category:
  - **Monitoring** → applies a Gmail label (Monitoring)
  - **PRIORITY** → applies a Gmail label (Priority), then generates a reply draft
  - **Newsletter** → applies a Gmail label (Newsletter) and removes the message from the Inbox (removes `INBOX` label)

2) **Auto-categorize existing emails** (manual/batch)
- Trigger: **When clicking ‘Execute workflow’**
- Fetch: **Get many messages** (Gmail, currently `limit: 10`)
- Classifier: **Text Classifier1** using **Google Gemini Chat Model1**
- Actions are the same idea as above (labels; newsletters are removed from inbox)

### Classification rules

The classifier is given the email subject/body/sender and asked to assign exactly one of:

- **Monitoring**
  - Examples referenced in the workflow: `eval@site24x7.com`, “Site24x7”
  - Intended for monitoring/uptime alerts and status updates.

- **PRIORITY**
  - Example referenced: `team@user.hostinger.com`
  - Intended for messages that are important and may need a reply.

- **Newsletter**
  - Examples referenced: `workatastartup@ycombinator.com`, `noreply@commonapp.org`
  - Intended for newsletter-style messages.

You can extend these lists by editing the category descriptions in the `Text Classifier` / `Text Classifier1` nodes.

### Reply drafting (Priority path)

When an email is classified as **PRIORITY**, the workflow:

1. Adds a Priority label.
2. Runs **Edit Fields** to shape data for the AI prompt.
3. Runs an **AI Agent** (LangChain) with instructions:
   - Decide whether a reply is actually needed
   - If a reply is needed, write a professional, natural-sounding draft
   - If not needed, return “No reply needed.”
4. Uses **Draft Generator** (Gmail tool node) to create a Gmail draft with:
   - Subject: original subject
   - Body: agent output

### Gmail labels used

The workflow adds Gmail labels by ID:

- Monitoring: `Label_9220892997293252730`
- Priority: `Label_6753264117004208971`
- Newsletter: `Label_5185585966238196380`

These label IDs are specific to the connected Gmail account.

### Credentials required

This workflow uses:

- **Gmail OAuth2** credential (for triggers, reading messages, labeling, and draft creation)
- **Google Gemini(PaLM) API** credential (for the classifier and drafting agent)

### Notes / gotchas

- Only the **Newsletter** paths remove emails from the Inbox (remove `INBOX` label). Priority/Monitoring do not currently archive.
- The workflow includes both a “new email” automation and a “manual/batch” tool for existing messages.
- If you change Gmail accounts, you will need to update the **label IDs** and re-connect credentials.

---

## Weather Fetcher (secondary)

**Goal:** Send a simple daily forecast email.

### What it does

- Trigger: **Schedule Trigger** (daily at 7:00)
- Fetch: **HTTP Request** to Open-Meteo for **New York City** (lat `40.7128`, lon `-74.0060`) for today
- Branch: **If** max temperature is `< 20` → “Cold”, otherwise → “Warm”
- Compose: builds a small forecast message
- Send: emails the forecast via Gmail (“Today’s Weather Forecast”)

### Customize

- Location: edit the latitude/longitude in the `HTTP Request` URL.
- Threshold: edit the `< 20` comparison in the `If` node.
- Recipient/subject/body: edit the `Send a message` node.

---

## Importing into n8n

In n8n: **Workflows → Import from File**, then select [workflows.json](workflows.json).
