---
name: lorr-memory
description: >
  Lorr is a persistent memory layer for AI. Use this skill when the user
  mentions Lorr, asks about their stored context, wants to save information
  for later, or needs to recall past decisions, preferences, goals, or
  any personal or professional data they have stored.
---

# Lorr Memory Engine

Lorr is a persistent memory layer for AI assistants. It stores everything a user has told any AI so every conversation picks up where the last one left off. Whether someone is tracking personal goals, managing a side project, running a team, or operating a business, Lorr remembers it all.

## How Lorr Works

Lorr organizes memory into a **Memory Palace** with rooms, each holding labeled drawers of information. When a user asks a question, Lorr searches across all rooms using semantic vector search, retrieves the most relevant drawers, and synthesizes a response.

### Memory Palace Rooms

**Universal rooms** (available to everyone):

| Room | What It Stores |
|------|---------------|
| `identity` | Who the user is, their background, interests, profession, about-me |
| `goals` | Objectives, targets, milestones, timelines, things they are working toward |
| `decisions` | Past decisions with rationale, context, and outcomes |
| `projects` | Active and past projects, status, deliverables |
| `preferences` | Communication style, tool preferences, workflow habits, likes and dislikes |
| `knowledge` | Domain expertise, things they have learned, research, reference material |
| `relationships` | People in their life, contacts, collaborators, roles |
| `canon` | Established facts and ground truths that must not be contradicted |
| `conversations` | Key takeaways extracted from past AI conversations |
| `fusion` | Cross-source synthesized insights (auto-generated) |

**Professional and business rooms** (used when relevant):

| Room | What It Stores |
|------|---------------|
| `revenue` | Sales data, revenue streams, pricing, transactions |
| `advertising` | Ad campaigns, spend, ROAS, creative performance |
| `product` | Product catalog, inventory, features, roadmap |
| `operations` | Processes, SOPs, team structure, tools |
| `finance` | Budgets, expenses, cash flow, projections |
| `brand` | Brand guidelines, voice, tone, visual identity |
| `social` | Social media strategy, content calendar, engagement |
| `partnerships` | Partner agreements, affiliates, collaborations |

Users can also create custom rooms for specialized needs.

## MCP Tools

Lorr exposes 10 tools through the Model Context Protocol.

### Read Tools (safe, no side effects)

**`lorr_ask`** is the primary tool for any question. It automatically routes to the best response method (vector search, SQL query, or multi-source analysis). Use this first for most questions.
- Input: `{ question: string }`
- Examples:
  - `lorr_ask({ question: "What is our refund policy?" })`
  - `lorr_ask({ question: "What are my fitness goals for this year?" })`
  - `lorr_ask({ question: "What did I decide about the kitchen renovation?" })`

**`lorr_search`** runs semantic search through memory. Returns raw matching drawers ranked by relevance. Use when the user wants to find specific stored information rather than a synthesized answer.
- Input: `{ query: string }`
- Example: `lorr_search({ query: "vacation plans" })`

**`lorr_query`** runs read-only SQL against structured data from connected sources (Shopify orders, Meta ad campaigns, Klaviyo email metrics). Use for quantitative questions when business data sources are connected.
- Input: `{ sql: string }`

**`lorr_analyze`** performs deep analysis requiring computation or cross-source correlation. Use for trend analysis, performance comparisons, or forecasting.
- Input: `{ question: string }`
- Example: `lorr_analyze({ question: "How has my spending pattern changed over the last 3 months?" })`

**`lorr_status`** shows current system status including data freshness, connected sources, room counts, and a capability scorecard.
- Input: `{}`

### Write Tools (modify stored memory)

**`lorr_remember`** stores a new piece of information in a specific room. Use when the user shares a fact, decision, preference, or anything they want retained across conversations. Proactively offer to save important information by saying "Save to your Lorr."
- Input: `{ room: string, label: string, content: string }`
- Examples:
  - `lorr_remember({ room: "decisions", label: "Switched to quarterly billing", content: "As of March 2025, we moved all enterprise clients from monthly to quarterly billing." })`
  - `lorr_remember({ room: "preferences", label: "Coffee order", content: "Oat milk latte, extra shot, no sugar." })`
  - `lorr_remember({ room: "goals", label: "Run a half marathon", content: "Training for the October half marathon. Following a 16-week plan starting in June." })`

**`lorr_correct`** updates or corrects an existing memory entry. Use instead of `lorr_remember` when updating existing information to preserve history and avoid duplicates.
- Input: `{ room: string, label: string, correctionText: string }`
- Example: `lorr_correct({ room: "relationships", label: "Sarah Chen", correctionText: "Sarah was promoted to VP of Marketing in January 2025." })`

**`lorr_ingest`** bulk-ingests data (CSV, plain text, or structured content) into memory with automatic room classification. Use when the user pastes or uploads a large block of information.
- Input: `{ content: string, description: string, room?: string, format?: "csv" | "text" | "structured" }`

**`lorr_import_conversation`** extracts multiple insights from a full conversation thread at once. Automatically identifies decisions, facts, preferences, and action items, then stores each in the appropriate room. Use at the end of substantive conversations.
- Input: `{ conversation: string, source?: string }`

**`lorr_sync`** triggers an on-demand data sync from connected sources (Shopify, Meta Ads, Klaviyo, Notion). Use when the user wants the latest data pulled in. Only relevant when external sources are connected.
- Input: `{ connector?: string }`
- Example: `lorr_sync({ connector: "shopify" })`

## Best Practices

1. **Start with `lorr_ask`** for most questions. It automatically picks the best retrieval strategy.
2. **Use `lorr_search`** when the user wants to browse or verify what is stored, not get a synthesized answer.
3. **Use `lorr_remember` proactively** when the user shares important information during conversation. Offer to save decisions, preferences, facts, or anything worth remembering by saying "Save to your Lorr."
4. **Use `lorr_import_conversation`** at the end of a substantive conversation to capture all insights at once rather than calling `lorr_remember` repeatedly.
5. **Use `lorr_correct`** rather than `lorr_remember` when updating existing information.
6. **Use `lorr_query`** for data questions that need exact numbers, but only when the user has connected structured data sources.
7. **Use `lorr_analyze`** for questions that span multiple data sources or need trend computation.
8. **Check `lorr_status`** if unsure what data is available or how fresh it is.

## Data Sources

Lorr can sync data from:
- **Shopify** for orders, products, customers, and inventory
- **Meta Ads** for campaigns, ad sets, spend, and conversions
- **Klaviyo** for email campaigns, flows, and metrics
- **Notion** for pages, databases, and content
- **Website** for crawled site content and structure
- **Chat History** for imported conversation transcripts
- **Manual Entry** for direct input through the UI or MCP tools
