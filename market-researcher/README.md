# Market Researcher

An autonomous competitive intelligence agent that monitors your competitors weekly, researches what they've been up to, and emails you a structured report.

This is the most interesting agent in the repo. Not because the final version is especially complex, but because the JSON file contains four iterations of the same idea. Each one taught me something about how agents work (and don't work) in practice.

## What it does

Every week, on a schedule:

1. Reads a list of competitors from a Google Sheet
2. Researches each competitor using Perplexity (funding, product launches, partnerships, user sentiment)
3. Synthesizes the findings into a structured report
4. Emails the report to you

No manual work. You maintain the competitor list in a spreadsheet, and the agent handles the rest.

## The evolution

The JSON file has four versions of this agent stacked in one workflow. Here's what happened with each one and why I changed it.

### v1: One agent does everything

The first version was a single AI agent connected to Google Sheets, Perplexity, and Gmail. The prompt told it to read the competitor list, research each one, write a report, and email it.

It worked. Sometimes. The agent would research two out of five competitors and call it done. Or it would spend all its tokens on one competitor and have nothing left for the others. The report quality was inconsistent. Some weeks you'd get a thorough breakdown, other weeks you'd get three sentences.

The problem: one agent doing four different jobs (data retrieval, research, synthesis, delivery) with no structure around the order of operations. It had too much freedom and not enough accountability.

### v2: Better prompting, better model

I switched from OpenAI to Claude Sonnet 4.5 and rewrote the prompt with explicit search patterns. Instead of "research this competitor," the prompt specified three concrete searches per competitor:

- Official news (BusinessWire, PR Newswire)
- Product and partnership announcements
- Industry coverage (TechCrunch, Reddit, etc.)

The output format was locked down too. Structured JSON, specific fields, no room for the agent to improvise.

This was better. The research was more thorough and the output was more consistent. But the token usage was still high because one agent was holding the entire context of all competitors, all searches, and the final report in a single conversation. For 5-10 competitors with 3 searches each, that adds up fast.

### v3: Parallel specialized agents

The breakthrough idea: split the work into specialized agents.

- A News Research Agent that only finds funding, products, and partnerships
- A Sentiment Research Agent that only finds user feedback and complaints
- A Synthesis Agent that takes both outputs and writes the final report

The News and Sentiment agents run in parallel from the same input (the competitor list), then their outputs feed into the Synthesis Agent, which formats the report and sends the email.

Each agent has a focused prompt, a clear contract for its input/output format (JSON arrays), and a limited scope. Token usage dropped because each agent only carries context for its specific task.

The problem: n8n doesn't handle parallel agent completion cleanly. If one agent finishes before the other, the Synthesis Agent gets triggered with incomplete data. The fan-in pattern isn't natively supported in n8n's execution model the way you'd want it to be.

### v4: Sequential specialized agents

Same architecture as v3, but sequential instead of parallel. News Agent runs first, finishes, then Sentiment Agent runs, finishes, then the data goes to a synthesis step (using a structured OpenAI call instead of a full agent), then the email sends.

It's slower. A full run takes longer because nothing happens in parallel. But it's reliable. Every step completes before the next one starts. The data is always complete when it reaches synthesis.

This is the version that runs in production.

## What I learned

Three things came out of iterating on this:

1. Single agents doing multiple complex tasks will cut corners. They lose track of where they are, skip steps, and produce inconsistent results. Splitting work across specialized agents with clear input/output contracts fixes this.

2. The Think tool matters more than you'd expect. Agents without a scratchpad tend to lose context between tool calls. The Think tool gives them a place to track progress, and the difference in completion rates is noticeable.

3. Sequential beats parallel when your orchestrator doesn't support proper fan-in. Parallel is faster in theory, but if your system doesn't wait for all branches to complete before proceeding, you get partial results. Take the speed hit and run things in order.

## Setup

1. Create a Google Sheet with a column called "Competitor Name" and list your competitors
2. Import `market-researcher.json` into n8n
3. Connect your Google Sheets account (OAuth2)
4. Connect your Gmail account (OAuth2)
5. Connect your Perplexity API key
6. Connect your OpenAI API key
7. Update the Google Sheet reference in the workflow to point to your spreadsheet
8. Update the email address in the Gmail nodes to your own
9. Set the schedule trigger to your preferred frequency (default is weekly at 6am)
10. Activate the workflow

The JSON has all four versions in it. The active sequential version (v4) is the bottom section of the workflow. The others are there for reference but aren't connected to the triggers.

## Integrations

- Google Sheets (competitor list input)
- Perplexity API (research)
- OpenAI API (agent reasoning, synthesis)
- Gmail (report delivery)
- Anthropic API (used in some iterations)
