# Korean Care RAG Chatbot

AI customer service chatbot for **koreancare.lt** -- a Korean skincare e-commerce store. Built with RAG for product knowledge, skincare routine recommendations, and WooCommerce order tracking.

## What It Does

- Answers product questions (ingredients, skin types, routines) using RAG from Supabase vector store
- Builds personalized Korean-style skincare routines (cleanser, toner, serum, cream, SPF)
- Tracks WooCommerce orders with three-step verification (order number + email + amount)
- Auto-updates knowledge base when Google Drive source file changes
- Maintains conversation memory per session via PostgreSQL
- Logs all conversations to Google Sheets

## Architecture

```
Storefront Chat Widget
        |
    Webhook
        |
    AI Agent (GPT-4.1-mini)
    |       |               |
    |       |               Postgres Chat Memory (per conversation)
    |       |
    |       Order_Status
    |       (HTTP Request -> WooCommerce API)
    |
    korean_care_kb
    (Supabase Vector Store + OpenAI Embeddings)
    Products | Routines | FAQ | Policies
        |
    Edit Fields
    |           |
    Reply to    Google Sheets
    Widget      Logging

--- Ingestion Pipeline (separate workflow) ---

Google Drive Trigger -> Delete Old Docs -> Download File -> Supabase Vector Store (insert)
                                                            |
                                                        OpenAI Embeddings
```

## Workflows

| File | Description | Nodes |
|------|-------------|-------|
| `Korean_Care_RAG.json` | Main chatbot -- RAG agent with skincare knowledge, order tracking, conversation memory | 9 |
| `Korean_Care_Order_Status.json` | Order status API proxy -- forwards requests to WooCommerce with secret header auth | 2 |
| `Koreancare_UpdateKnowledgebase.json` | Knowledge base ingestion -- watches Google Drive, clears old vectors, re-embeds | 6 |

## Tech Stack

- **n8n** -- workflow automation
- **OpenAI GPT-4.1-mini** -- chat model
- **Supabase** -- vector store for RAG
- **PostgreSQL** -- conversation memory
- **WooCommerce API** -- order tracking
- **Google Drive** -- knowledge base source files
- **Google Sheets** -- conversation logging

## Key Features

- **Korean-style routine builder** -- 10-step routine logic (cleanser, toner, essence, serum, ampoule, sheet mask, eye cream, moisturizer, SPF, sleeping mask)
- **Three-step order verification** -- order number + email + exact amount before revealing status
- **Auto-refresh knowledge base** -- Google Drive file changes trigger automatic re-embedding
- **73 top-K retrieval** -- comprehensive product coverage for accurate answers
- **Bilingual** -- Lithuanian by default, auto-detects and responds in English
- **Formal Lithuanian** -- uses "Jus" form throughout

## Required Credentials

| Credential | Purpose |
|------------|---------|
| OpenAI API | GPT-4.1-mini chat model + embeddings |
| Supabase API | Vector store for product knowledge |
| PostgreSQL | Conversation memory storage |
| WooCommerce API secret | Order status verification (header auth) |
| Google Drive OAuth2 | Knowledge base source file monitoring |
| Google Sheets OAuth2 | Conversation logging |

## Setup

1. **Import workflows** -- import all three `.json` files into your n8n instance
2. **Configure credentials** -- set up all credentials listed above in n8n
3. **Set up Supabase** -- create a vector store table with appropriate schema and enable pgvector extension
4. **Configure WooCommerce** -- set the shared secret header for order status API authentication
5. **Prepare knowledge base** -- upload your product/FAQ/policy documents to Google Drive
6. **Run ingestion** -- trigger the knowledge base workflow to embed initial documents
7. **Connect chat widget** -- point your storefront chat widget to the webhook URL
8. **Activate workflows** -- enable the main chatbot and the Google Drive trigger workflow
