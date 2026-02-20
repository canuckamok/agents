# Voice Assistant

The same Ash personal assistant from the [Personal Assistant](../personal-assistant/) agent, but accessible over Telegram. Text or voice. You send a voice memo while walking the dog, and it texts you back with your calendar summary.

## The problem

The chat-triggered version of Ash works fine when you're at your desk. But sometimes you're driving, or cooking, or doing anything where typing isn't happening. You still want to ask "do I have anything at 3pm?" without pulling up a browser.

Telegram solves the interface problem. It's already on your phone, it handles voice natively, and it delivers responses as push notifications.

## What it does

Send Ash a Telegram message (text or voice). If it's voice, the agent transcribes it with OpenAI Whisper, then processes the question through the same email/calendar agent. The response comes back as a Telegram message.

Everything the Personal Assistant does, this does too. Same capabilities, same guardrails. The difference is the input/output channel.

## How it works

The flow has a branching pattern:

1. Telegram Trigger receives a message
2. A Switch node checks: does this message have a voice attachment?
3. If voice: download the audio file from Telegram, send it to OpenAI Whisper for transcription, pass the text to the agent
4. If text: extract the message text directly, pass it to the agent
5. The agent does its thing (same Plan/Execute/Update loop, same tools)
6. Response goes back to the user's Telegram chat

The Switch node is the key design decision. Without it, you'd need separate workflows for text and voice, or a single path that tries to transcribe everything (including text messages, which would fail). The branch-and-merge pattern keeps it clean.

Memory is keyed to the Telegram chat ID, so each user gets their own conversation history.

Components:
- Telegram Trigger
- Switch node (voice vs. text detection)
- Telegram file download (for voice messages)
- OpenAI Whisper transcription
- OpenAI GPT-4o-mini agent with Think, Gmail, and Calendar tools
- Buffer window memory (keyed per chat)
- Telegram send message (response)

## Setup

1. Create a Telegram bot via [BotFather](https://t.me/botfather). Save the API token.
2. Import `voice-assistant.json` into n8n
3. Connect your Telegram API credentials (the token from step 1)
4. Connect your Gmail account (OAuth2)
5. Connect your Google Calendar account (OAuth2)
6. Connect your OpenAI API key (used for both the agent and Whisper)
7. Activate the workflow
8. Send your bot a message on Telegram to test

## Design decisions

Why Telegram and not WhatsApp or SMS? Telegram has a free, well-documented bot API. WhatsApp requires a business account and Meta approval. SMS requires a paid service like Twilio. Telegram is the path of least resistance for a personal tool.

Why Whisper for transcription? It's accurate, it handles accents well, and since we're already using the OpenAI API for the agent, it's one less credential to manage.
