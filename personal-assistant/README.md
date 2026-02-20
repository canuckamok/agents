# Personal Assistant

An AI agent named Ash that reads your Gmail and Google Calendar, then answers questions about them.

## The problem

You're in the middle of something and need to know if that meeting got rescheduled, or what the last email from your manager said, or whether you have anything before 10am tomorrow. The normal move is to open Gmail, get distracted by 14 unread threads, lose 20 minutes, and forget why you opened it in the first place.

Ash handles the lookup so you don't have to context-switch.

## What it does

You ask Ash a question in natural language. "What's on my calendar today?" or "Summarize the important emails from this week." Ash searches your Gmail and Calendar, filters out marketing junk (anything with an unsubscribe link gets ignored), and gives you a clean answer with links to the original emails.

It remembers the conversation, so you can ask follow-ups.

## What it won't do

- Send emails. It reads and drafts, but never sends.
- Make things up. If it doesn't have the data, it tells you.
- Include marketing emails in summaries. If the email has "unsubscribe" in the footer, it gets filtered out.
- Decline calendar meetings.

These guardrails are baked into the system prompt. You can adjust them, but they're there for a reason.

## How it works

The agent runs a strict Plan/Execute/Update loop:

1. Receives your question
2. Creates a plan using the Think tool (its working memory/scratchpad)
3. Calls Gmail and/or Calendar tools to get the data
4. Updates the plan based on what it found
5. Responds with a structured summary

The Think tool is the interesting piece here. It gives the agent a place to reason between tool calls. Without it, the agent tends to fire off searches without tracking what it's already found or what's still missing.

Components:
- Chat trigger (n8n's built-in chat interface)
- OpenAI GPT-4o-mini (fast, cheap, good enough for this task)
- Buffer window memory (keeps conversation context)
- Think tool (agent scratchpad)
- Gmail tool (read-only)
- Google Calendar tool (read-only)

## Setup

1. Import `personal-assistant.json` into n8n
2. Connect your Gmail account (OAuth2)
3. Connect your Google Calendar account (OAuth2)
4. Connect your OpenAI API key
5. Test it in the n8n chat interface

The model is set to GPT-4o-mini with temperature 0.3. You can swap in a different model if you want, but this one works well for structured retrieval tasks and keeps costs low.

## What I'd improve

- Add the ability to create calendar events (with explicit permission checks)
- Connect to Slack for another input channel
- Add a daily morning briefing on a schedule instead of waiting for a question
