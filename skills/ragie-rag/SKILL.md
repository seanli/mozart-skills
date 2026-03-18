---
description: "Retrieve content from Ragie.ai RAG platform using the retrieval API"
---

# Ragie RAG

Retrieve relevant document chunks from your Ragie.ai knowledge base. Use this when you need to answer questions or perform tasks that require information from uploaded documents, PDFs, coding docs, API references, or other content stored in Ragie.

## When to Use

- User asks about content from uploaded documents, PDFs, reports, or knowledge bases
- Questions about internal/proprietary information not available on the public web
- Looking up coding documentation, API references, or framework guides uploaded to the knowledge base
- Research tasks that require synthesizing information from multiple uploaded sources
- Any question where the answer likely lives in your Ragie knowledge base

## When NOT to Use

- General knowledge questions already in your training data
- Current events or live information (use web_search + web_fetch)
- Information you already have in memory (check search_memory first)

## Cross-check with your own knowledge

When RAG returns coding docs, API references, or technical content, use your training knowledge as a validation layer:
- Verify that API signatures, parameter names, and return types match what you know
- Flag if a retrieved snippet looks outdated or contradicts well-known patterns
- Fill in gaps — if RAG returns a partial example, complete it with your understanding
- When RAG and your knowledge conflict, prefer RAG for project-specific conventions but prefer your training data for language/framework fundamentals

## Quick Start

Use the `rag_retrieve` tool directly — it handles authentication and formatting:

    rag_retrieve: { query: "quarterly revenue figures", rerank: true }

For advanced usage (metadata filters, partitions, document listing), use shell + curl as shown below.

## Authentication

Your API key is available as `$RAGIE_API_KEY` in the environment. All requests use Bearer token auth.

## Advanced: Retrieving Content with curl

Use `shell` with `curl` to query the retrieval API directly. This gives access to metadata filters and other advanced parameters:

```bash
curl -s https://api.ragie.ai/retrievals \
  -H "Authorization: Bearer $RAGIE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "your search query here",
    "rerank": true
  }' | jq '.scored_chunks[:3]'
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `query` | string | required | The semantic search query |
| `rerank` | boolean | false | LLM reranking for higher accuracy (slower) |
| `top_k` | integer | 8 | Max chunks to return |
| `max_chunks_per_document` | integer | unlimited | Limit chunks per source document for diversity |
| `filter` | object | none | Metadata filter to scope results |

### Filtering by Metadata

Scope results to specific documents using metadata filters:

```bash
curl -s https://api.ragie.ai/retrievals \
  -H "Authorization: Bearer $RAGIE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "quarterly revenue",
    "rerank": true,
    "filter": {
      "department": {"$in": ["finance", "sales"]}
    }
  }' | jq '.scored_chunks[:5]'
```

Supported filter operators: `$eq`, `$ne`, `$gt`, `$gte`, `$lt`, `$lte`, `$in`, `$nin`. Combine with `$and` / `$or`.

### Response Format

The API returns `scored_chunks`, each containing:

```json
{
  "text": "The actual chunk content...",
  "score": 0.95,
  "id": "chunk-uuid",
  "document_id": "doc-uuid",
  "document_name": "Q4 Report.pdf",
  "document_metadata": { "department": "finance" }
}
```

## Discover Available Content

Before retrieving, understand what's in the knowledge base.

### List Documents

```bash
curl -s https://api.ragie.ai/documents \
  -H "Authorization: Bearer $RAGIE_API_KEY" \
  | jq '.documents[] | {name, id, status}'
```

Returns all uploaded documents with their names, IDs, and processing status. Use this to see what's available before writing queries.

### Get Document Summary

```bash
curl -s "https://api.ragie.ai/documents/DOCUMENT_ID/summary" \
  -H "Authorization: Bearer $RAGIE_API_KEY" \
  | jq '.summary'
```

Returns an LLM-generated overview of a specific document. Use this to understand a document's scope and key topics before doing targeted chunk retrieval.

## Research and Write Workflow

When using Ragie content to produce written output (articles, reports, summaries), follow this workflow:

### 1. Discover

List documents and read summaries to understand what source material is available:

```bash
# See all documents
curl -s https://api.ragie.ai/documents \
  -H "Authorization: Bearer $RAGIE_API_KEY" \
  | jq '.documents[] | {name, id}'

# Read the summary of a specific document
curl -s "https://api.ragie.ai/documents/DOCUMENT_ID/summary" \
  -H "Authorization: Bearer $RAGIE_API_KEY" \
  | jq '.summary'
```

### 2. Plan Queries

Break the writing topic into 3-5 targeted retrieval queries. Specific queries retrieve better chunks than broad ones.

Example — writing about Newton's contributions:
- "Newton's three laws of motion"
- "Newton's development of calculus"
- "Key arguments in Principia Mathematica"
- "Newton's work on optics and light"
- "Newton's influence on scientific method"

### 3. Retrieve

Run each query with `rerank: true` for accuracy. Set `max_chunks_per_document` to pull from multiple sources:

```bash
curl -s https://api.ragie.ai/retrievals \
  -H "Authorization: Bearer $RAGIE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "Newton three laws of motion",
    "rerank": true,
    "top_k": 5,
    "max_chunks_per_document": 2
  }' | jq '.scored_chunks[] | {text, score, document_name}'
```

### 4. Synthesize

Combine retrieved chunks into a coherent draft. Always cite the source document for each key claim:

> Newton formulated three laws of motion that became the foundation of classical mechanics (Source: Principia.pdf). The first law states that... (Source: Newton_Biography.pdf)

### 5. Fill Gaps

After drafting, identify missing information and run follow-up queries. Iterate until the content is complete. Save key facts to memory for reuse:

```
save_to_memory: "Newton's Principia was first published in 1687 and contained three books (from Principia.pdf)"
```

## Source Citation

Always track and cite the `document_name` field from retrieved chunks. This ensures the generated content is traceable back to its source material.

**Inline citations** — Reference the source after each key claim:

> The experiment demonstrated a 40% improvement in efficiency (Source: Lab_Results_Q4.pdf).

**References section** — When producing longer content, collect all cited documents into a references list at the end:

> **References**
> - Principia.pdf — Newton's original work on mechanics and gravitation
> - Newton_Biography.pdf — Biographical context and historical significance
> - Optics_Letters.pdf — Newton's correspondence on light experiments

**Score-based confidence** — Chunks with `score > 0.8` are strong matches. For scores below `0.5`, consider the information less reliable and verify with additional queries.

## Partition Scoping

If documents are organized into partitions (by project, topic, or team), scope retrieval to a specific partition using the `partition` header:

```bash
curl -s https://api.ragie.ai/retrievals \
  -H "Authorization: Bearer $RAGIE_API_KEY" \
  -H "Content-Type: application/json" \
  -H "partition: newton-research" \
  -d '{
    "query": "laws of motion",
    "rerank": true
  }' | jq '.scored_chunks[:5]'
```

The `partition` header also works with `GET /documents` to list only documents in a specific partition.

## Usage Guidelines

- **Always use `rerank: true`** when the retrieved content will be used for generation — it reduces hallucinations significantly
- **Use `rerank: false`** for speed-sensitive lookups or when you're scanning broadly
- **Pipe through `jq`** to extract just the text: `| jq -r '.scored_chunks[].text'`
- **Set `max_chunks_per_document`** when you want diversity across multiple source documents
- **Combine with memory**: Save important retrieved facts to `save_to_memory` so you don't re-query for the same information

## Full API Reference

For the complete Ragie API (uploading documents, managing connections, partitions, entity extraction, etc.), fetch the docs:

```
web_fetch https://docs.ragie.ai/llms.txt
```

This returns a table of contents linking to all API reference pages. Fetch individual pages as needed with `web_fetch`.
