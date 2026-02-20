# Tweet RAG

Two workflows that work together. One ingests tweets from accounts you care about into a searchable database. The other lets you ask questions against that database in plain English.

## The problem

You follow smart people on Twitter. Product thinkers, founders, engineers. They post things worth reading. You miss most of it because the timeline is a firehose and the algorithm has its own priorities.

You could scroll more. Or you could build a system that collects the signal, stores it, and lets you search it on your own terms.

## How it works

This is a RAG (Retrieval-Augmented Generation) system split into two independent workflows.

### Workflow 1: Ingestion (`ingestion-flow.json`)

Runs on a schedule (or manually). Here's the pipeline:

1. Reads a list of Twitter accounts from a Google Sheet (handles, names, descriptions)
2. Pulls the last 10 tweets from each account via the Twitter/X API
3. Loops through each account one at a time to stay within rate limits
4. Cleans the data: removes retweets, drops tweets shorter than 10 characters, deduplicates by tweet ID
5. Forks the cleaned data into two parallel paths:
   - **Vector store path**: formats each tweet with author context, converts it to a vector embedding using Cohere, and inserts it into a Supabase vector database with metadata (author, engagement metrics, URL, timestamp)
   - **Sheets path**: appends a structured row to a Google Sheet with the raw tweet data and metrics (likes, retweets, replies, quotes, impressions)

The vector embedding is the part that makes semantic search work. It converts the meaning of each tweet into a numerical fingerprint, so "AI is coming for PM jobs" and "will product managers be automated away" end up near each other in the database even though they share zero keywords.

### Workflow 2: Chat (`chat-flow.json`)

A chat interface where you ask questions and an AI agent searches your tweet archive to answer them.

The agent has two tools:

- **Vector search** (Supabase): for semantic, meaning-based queries. "What are people saying about AI and product management?" finds relevant tweets regardless of exact wording.
- **Google Sheets**: for structured, metric-based queries. "What were the most-liked tweets this week?" or "Show me everything from @shreyas" goes here because it needs to sort and rank by numbers.

The agent decides which tool to use based on your question. Concept questions go to the vector store. Ranking questions go to the sheet. It uses the Think tool as a scratchpad to plan its approach and track what it's found.

Every answer includes the original author, a link to the tweet, and engagement context when relevant.

## Why two workflows instead of one

Ingestion is a batch job. It runs on a schedule, pulls data, and writes to storage. Retrieval is interactive. You ask a question when you have one.

Coupling them would mean the chat agent triggers ingestion, or ingestion blocks on chat availability. Neither makes sense. Separation keeps each workflow simple and independently testable.

## Setup

### Prerequisites

- An n8n instance
- A Twitter/X API developer account (for tweet access)
- A Supabase project with a vector-enabled table called "documents"
- A Cohere API key (for embeddings)
- An OpenAI API key (for the chat agent)
- A Google account (for Sheets)

### Ingestion workflow

1. Create a Google Sheet with columns: Handle, Name, About. Add the Twitter accounts you want to track.
2. Create a second Google Sheet for storing tweet data (the workflow will append rows with: tweet_id, tweet_text, tweet_url, author_username, author_name, likes, retweets, replies, quotes, impressions)
3. Set up a Supabase project and create a vector-enabled table called "documents"
4. Import `ingestion-flow.json` into n8n
5. Connect your credentials: X/Twitter API, Google Sheets (OAuth2), Cohere API, Supabase API
6. Update the Google Sheet references to point to your spreadsheets
7. Run it manually first to verify, then set the schedule trigger

### Chat workflow

1. Import `chat-flow.json` into n8n
2. Connect your credentials: OpenAI API, Cohere API (same embedding model as ingestion), Supabase API, Google Sheets (OAuth2)
3. Update the Google Sheet reference to point to your tweets spreadsheet
4. Open the n8n chat interface and ask a question

The embedding model in both workflows needs to match. Both use Cohere's `embed-english-light-v2.0`. If you change one, change the other, or your searches won't return results.

## Integrations

- Twitter/X API (tweet ingestion)
- Google Sheets (account list input, tweet data storage, structured queries)
- Cohere API (vector embeddings)
- Supabase (vector database for semantic search)
- OpenAI API (chat agent reasoning)
