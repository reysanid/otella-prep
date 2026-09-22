# FAQ Semantic Search — n8n AI Agent

*[نسخه فارسی](README.fa.md)*

## Overview
An AI agent for semantic search over FAQs from a Design Thinking course. Users submit a question through a form, and the agent retrieves the most relevant answer by searching stored embeddings.

## Pipeline

**Ingestion:**
Read/Write Files from Disk (faq.csv)
→ Extract from File (CSV)
→ Embeddings Ollama
→ Simple Vector Store (Insert Documents)

**Retrieval:**
On new n8n form event (user question)
→ Embeddings Ollama
→ Simple Vector Store (Get Many, limit 4)
→ Edit Fields (extract final answer)

## Stack
- **n8n** — running on Docker (Windows)
- **Ollama** (`nomic-embed-text` model) — used for embeddings instead of OpenAI, due to international access restrictions
- **In-memory Vector Store** — data is lost on container restart, so ingestion must be re-run after each restart

## Challenges & Debugging
Built this agent while debugging four real issues:

1. **Broken ingestion connection** — the `Extract from File` node wasn't wired to `Simple Vector Store` on canvas; connected it manually.
2. **Mismatched Memory Keys** — ingestion used `faq_store` while retrieval used a different key; aligned them.
3. **Metadata typo** — `{{ json.answer$ }}` needed to be `{{ $json.answer }}` — fixed, and metadata.answer resolved correctly.
4. **Wrong path in Edit Fields** — `{{ $json.output[0].metadata.answer }}` needed to be `{{ $json.document.metadata.answer }}`, matching the actual retrieval output structure.

## Status
Fully tested end-to-end — submitting a question through the form returns the correct answer.
