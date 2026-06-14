# AI Knowledge Assistant (n8n + Supabase + Gemini)

An AI-powered Knowledge Assistant built with **n8n workflows**, **Supabase**, and **Google Gemini**.

This project lets users ingest document text, store vector embeddings, and ask natural language questions against the knowledge base with **source-backed answers** and **conversation history**.

---

## Overview

This project is a **RAG-based (Retrieval-Augmented Generation)** knowledge assistant designed as a practical MVP.

It supports:

- Document text ingestion
- Chunking and embedding generation
- Vector search over stored chunks
- Citation-aware responses
- Multi-turn conversations
- Persistent chat history
- n8n-based orchestration
- Supabase-backed storage

The backend logic is implemented as **n8n workflows** instead of a traditional code server.

---

## Features

### Core Features
- Upload or ingest document text
- Split documents into semantic chunks
- Generate embeddings using **Gemini Embeddings**
- Store chunks and metadata in **Supabase**
- Ask questions in natural language
- Retrieve relevant chunks using vector search
- Generate answers with inline citation references like `[S1]`
- Store conversations and messages for future turns

### Optional Extras
- Simple HTML frontend
- Postman collection for API testing
- curl / PowerShell test commands
- File/PDF ingest workflow template via external parser API

---

## Tech Stack

### Workflow / Backend
- **n8n**

### Database / Storage
- **Supabase**
- **PostgreSQL**
- **pgvector**

### AI / Models
- **Google Gemini API**
  - `gemini-embedding-2` for embeddings
  - `gemini-2.5-flash` / Gemini OpenAI-compatible endpoint for answer generation

### Testing / Client Tools
- **Postman**
- **curl / PowerShell**

### Frontend
- Simple static **HTML/CSS/JS** interface

---

## Architecture

```text
User / Frontend
      |
      v
   n8n Webhooks
      |
      +--> Ingest Workflow
      |      - receive document text
      |      - chunk text
      |      - generate embeddings with Gemini
      |      - save document + chunks to Supabase
      |
      +--> Chat Workflow
             - receive user question
             - create query embedding
             - retrieve matching chunks from Supabase
             - build prompt with citations
             - generate answer with Gemini
             - save assistant reply + conversation history
```

---

## Workflows Included

### 1. `ka-mvp-ingest-text.workflow.json`
Text ingestion workflow.

**Purpose:**
- Accept a document as raw text
- Chunk the text
- Generate embeddings
- Save document and chunks in Supabase

**Input:**
```json
{
  "title": "HR Policy",
  "file_name": "hr-policy.txt",
  "text": "Employees are entitled to 20 days of annual leave..."
}
```

**Output:**
```json
{
  "status": "ready",
  "document_id": "...",
  "title": "HR Policy",
  "file_name": "hr-policy.txt",
  "message": "Document indexed successfully",
  "chunk_count": 1,
  "summary": "Indexed 1 chunk(s)"
}
```

---

### 2. `ka-mvp-chat.workflow.json`
Chat workflow with retrieval and citations.

**Purpose:**
- Accept a user question
- Generate query embedding
- Search relevant chunks using vector similarity
- Build a grounded prompt
- Generate a source-backed answer
- Save messages to conversation history

**Input:**
```json
{
  "message": "How many annual leave days are employees entitled to?"
}
```

**Output:**
```json
{
  "conversation_id": "...",
  "answer": "Employees are entitled to 20 days of annual leave. [S1]",
  "citations": [
    {
      "source_id": "S1",
      "document_title": "HR Policy",
      "quote": "Employees are entitled to 20 days of annual leave..."
    }
  ]
}
```

---

## Database Schema

Supabase stores the following tables:

- `documents`
- `chunks`
- `conversations`
- `messages`

It also uses a vector similarity SQL function:

- `match_chunks(...)`

This is created by:

- `supabase-setup.sql`

---

## Project Structure

```text
knowledge-assistant-n8n/
├── README.md
├── ka-mvp-ingest-text.workflow.json
├── ka-mvp-chat.workflow.json
└── supabase-setup.sql
```

Related helper assets created for this project:

```text
knowledge-assistant-n8n-extras/
├── README.md
├── postman_collection.json
├── test-commands.md
├── ka-upgrade-ingest-file-template.workflow.json
└── frontend/
    └── index.html
```

---

## Setup Instructions

### 1. Create a Supabase project
Create a new project in Supabase and copy:

- Project URL
- Secret / service role key

### 2. Run the database setup SQL
Open the SQL editor in Supabase and run:

- `supabase-setup.sql`

This creates the required tables and vector search function.

### 3. Create a Gemini API key
Get a Gemini API key from Google AI Studio.

### 4. Import workflows into n8n
Import these workflows into separate n8n workflows:

- `ka-mvp-ingest-text.workflow.json`
- `ka-mvp-chat.workflow.json`

### 5. Configure credentials manually in the nodes
Update workflow nodes with:

- your Supabase project URL
- your Supabase secret key
- your Gemini API key

### 6. Publish the workflows
Once configured, publish both workflows in n8n.

### 7. Test the endpoints
Use Postman or curl to test:

- document ingestion
- chat question answering

---

## API Testing

A Postman collection and HTTP test commands were prepared for this project.

Use:

- `knowledge-assistant-n8n-extras/postman_collection.json`
- `knowledge-assistant-n8n-extras/test-commands.md`

---

## Frontend

A simple frontend is included at:

- `knowledge-assistant-n8n-extras/frontend/index.html`

This UI can be used to:

- ingest text documents
- send chat messages
- display assistant answers and citations

You can deploy it on:
- Netlify
- Vercel
- GitHub Pages

Then connect it to your two n8n production webhook URLs.

---

## How RAG Works in This Project

1. A document is sent to the ingest workflow.
2. The text is split into chunks.
3. Each chunk is embedded using Gemini.
4. Chunks are stored in Supabase with vectors.
5. A user asks a question.
6. The question is embedded using Gemini.
7. Supabase vector search retrieves the most relevant chunks.
8. The workflow builds a prompt using the retrieved sources.
9. Gemini generates the final answer.
10. Citations are extracted and stored with the assistant message.

---

## What I Used

This project was built using:

- **n8n** for workflow orchestration
- **Supabase** for relational data + vector storage
- **pgvector** for semantic retrieval
- **Google Gemini** for embeddings and answer generation
- **Postman** for endpoint testing
- **HTML/CSS/JavaScript** for a lightweight frontend

---

## Current Status

### Working
- Text document ingestion
- Chunk creation
- Gemini embeddings
- Supabase vector storage
- Retrieval workflow
- Citation-aware chat response generation
- Conversation + message persistence

### Optional Future Improvements
- PDF / DOCX parsing
- user authentication
- multi-user separation
- streaming responses
- reranking
- better source formatting
- memory summarization
- web search tools
- polished frontend dashboard

---

## Example Use Cases

- Policy assistant
- Internal documentation Q&A
- Notes / research assistant
- HR / legal knowledge assistant
- Product documentation bot

---

## Notes

- This project currently focuses on **text-based ingestion** for the MVP.
- File/PDF ingestion can be added using the included file workflow template plus an external parser API.
- Keep your **Supabase secret key** and **Gemini API key** private.
- Do not expose backend secrets in frontend code.

---

## License

Add your preferred license here, for example:

- MIT
- Apache-2.0
- Proprietary

---

## Author

Built as an AI Knowledge Assistant MVP using **n8n + Supabase + Gemini**.
