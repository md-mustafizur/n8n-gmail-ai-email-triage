# n8n Gmail AI Email Triage

An AI email triage workflow built with n8n, Gmail, Google Gemini and Telegram. It classifies incoming emails, labels them in Gmail, saves suggested replies as drafts, and sends an alert for urgent emails.

This is a practice project based on a real freelance job requirement. It was built and tested on a dedicated test Gmail account with dummy emails. No real client mailbox was used.

## What it does

- Picks up new emails automatically with a Gmail Trigger (checks every minute, up to 3 emails per check)
- Processes older emails on demand with a manual trigger (up to 7 emails per run, skipping emails that already have an AI label)
- Classifies each email into one of seven categories: Urgent, Important, Client, Billing, Routine, Informational, Low Value
- Applies the matching Gmail label (AI-Urgent, AI-Billing and so on)
- Writes a suggested reply and saves it as a Gmail draft when a reply is needed
- Sends a Telegram alert when an email is classified as Urgent
- Builds a category-wise summary report
- Uses an AI-NeedsReview label when the AI response is invalid

## Safety

- The workflow never sends emails. There is no Gmail Send node, only a Create a draft node. A human reviews every draft.
- Credentials are not included in this repository.

## How it works

1. Gmail Trigger (new emails) or Manual Trigger with Get many messages (old emails)
2. Edit Fields keeps only id, threadId, subject, from_email and text, so both paths share one format
3. Basic LLM Chain with Google Gemini returns JSON: category, priority, summary, suggested_reply, needs_human
4. Code node parses the JSON safely, and falls back to NeedsReview if the response is invalid
5. Get many labels and a Code node find the Gmail label ID for the category
6. Add label to message applies the label
7. If node: when suggested_reply is not empty, Create a draft saves the reply as a draft
8. If1 node: when the category is Urgent, a Telegram message is sent
9. A Code node builds a summary per category

## Setup

1. Import `Gmail_AI_Triage_Practice_clean.json` into n8n.
2. Create these Gmail labels: AI-Urgent, AI-Important, AI-Client, AI-Billing, AI-Routine, AI-Informational, AI-LowValue, AI-NeedsReview.
3. Connect your own credentials: Gmail OAuth2, Telegram and Google Gemini.
4. Replace the placeholders: `YOUR_SENDER_EMAIL@gmail.com` in the Get many messages filter and `YOUR_TELEGRAM_CHAT_ID` in the Telegram node.
5. Use a test Gmail account first.

## Testing

Tested with dummy emails covering urgent, billing, client, routine, informational and low value messages. Verified that labels are applied, drafts are created without any email being sent, Urgent emails trigger a Telegram alert, emails with no reply needed get no draft, and invalid AI output falls back to NeedsReview.

## What I learned

- Normalising data from two different triggers into one format
- Making draft creation depend on the reply text instead of an unreliable AI flag
- Handling invalid AI responses safely
