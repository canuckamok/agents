# AI Agents

A collection of AI agents I built in n8n. These are not demos or proof-of-concepts. They run on schedules, handle real data, and do work I used to do manually.

Each agent lives in its own folder with a README that explains what it does, how it works, and how to set it up yourself.

## The agents

| Agent | What it does |
|-------|-------------|
| [Personal Assistant](./personal-assistant/) | Reads your email and calendar, answers questions about them, drafts responses. Filters out marketing noise. |
| [Voice Assistant](./voice-assistant/) | Same assistant, but over Telegram. Send a voice message or text, get an answer back. |
| [Market Researcher](./market-researcher/) | Monitors your competitors weekly using Perplexity, synthesizes findings into a report, emails it to you. This one went through four architectural iterations. The README covers what worked, what didn't, and why. |
| [Tweet RAG](./tweet-rag/) | A two-part RAG system. One workflow ingests tweets from accounts you follow into a Supabase vector database. The other is a chat interface that lets you semantically search your archive. |

## Using these

1. Import the JSON file into your n8n instance
2. Swap in your own credentials (API keys, OAuth connections)
3. Run it

The JSON files have placeholder credentials. You need to bring your own API keys for OpenAI, Anthropic, Perplexity, Cohere, Supabase, Gmail, Google Calendar, Twitter/X, and/or Telegram depending on which agent you're setting up.

## About

Built by [Ilan Rotenberg](https://github.com/ilanrotenberg). Licensed under GPL v3.
