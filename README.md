# Important Email Interception

An n8n workflow that watches a Gmail inbox and sends a Telegram notification only for emails that actually need attention: replies to job applications, recruiter outreach, portfolio contact-form messages, client inquiries and personal emails. Newsletters, job alerts and automated notifications are ignored.

## How it works

1. **Schedule Trigger** runs every 10 minutes.
2. **Gmail: Get Many** fetches recent emails that haven't been processed yet (`-from:me -label:ai-checked newer_than:2d -category:promotions -category:social`).
3. **Loop Over Items** processes the emails one at a time.
4. **Basic LLM Chain** (Groq, `openai/gpt-oss-20b`) classifies each email and returns `important`, `category`, `reason` and `summary`.
5. Important emails are sent to Telegram with the sender, subject, summary and a direct Gmail link.
6. Every processed email gets the `ai-checked` Gmail label, so it is never classified twice. If a run fails, unlabeled emails are retried on the next run.

Errors are reported to a separate Telegram bot: classification failures from within the main workflow, and any other failure through the error workflow.

## Files

- `Important Email Interception.json`: the main workflow
- `Handle Errors and Notify via Telegram.json`: the error workflow

## Setup

1. Import both workflows into n8n.
2. Create credentials: Gmail OAuth2, Groq API, and a Telegram API credential for each bot (alerts and errors).
3. Create a Gmail label named `ai-checked` and select it in the **AddLabel** node.
4. Replace the placeholders below.
5. In the main workflow's **Settings**, set **Error Workflow** to the error workflow.
6. Activate the main workflow.

## Placeholders

| Placeholder | Where | What to put |
|---|---|---|
| `YOUR_TELEGRAM_CHAT_ID` | **Chat ID** in all Telegram nodes (both workflows) | Your Telegram chat ID. Send any message to your bot, then open `https://api.telegram.org/bot<TOKEN>/getUpdates` and copy `message.chat.id`. |
| `your-email@gmail.com` | **ClassifyEmail** system prompt and the Gmail link in **SendMessageToSuccessBot** | The Gmail address the workflow monitors. In the link, it makes the notification open the right account when you're signed in to several. |
| `YOUR_NAME` | **ClassifyEmail** system prompt | The user the workflow monitors. Also a part of a backup to filter out messages sent by user. |

You'll also want to adjust the system prompt in **ClassifyEmail** to describe who you are and what counts as important for you.