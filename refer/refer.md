# Challenge 1 - Build the Brain

## Overview

Set up the AI infrastructure that powers the enterprise knowledge platform. You will prepare document storage, deploy the AI models, build a vector search index from your document corpus, and wire it all together into a grounded AI agent that can answer policy questions with citations.

The challenge is complete when your Microsoft Foundry agent returns accurate, cited responses to enterprise knowledge queries in the playground.

---

## Prerequisites

Deploy all resources in the **same region** (recommended: sweden central, west us 2, ) before you start.

| Resource | Recommended SKU | Notes |
|---|---|---|
| Azure Storage Account | Standard LRS | You will create a blob container for the document corpus |
| Foundry Project | Standard | Create a Microsoft Foundry Project |
| Azure AI Search | Basic | Sufficient for this lab; enables semantic and vector search |

---

## Objectives

### 1. Document Storage

Set up the storage layer that holds your enterprise policy corpus.

- Create an Azure Storage Account and a blob container named `knowledge-docs`.
- Download the knowledge base dataset provided for this lab - 10 policy documents across HR, IT, and Finance categories.
- Upload all 10 documents to the `knowledge-docs` container.

---

### 2. AI Models

Prepare the models your agent and index will rely on.

- In your Microsoft Foundry project, deploy the following two models:
  - A chat completion model - use `gpt-4.1` or `gpt-4.1-mini` (whichever is available in your region).
  - An embedding model - use `text-embedding-ada-002`.
- Note the deployment names - you will need them when configuring the index and agent.

---

### 3. Knowledge Index

Build a vector search index from the documents in Blob Storage.

- Create an Azure AI Search instance (Basic SKU is sufficient to control cost).
- Import and vectorize your datasets and create an index.
- Once the indexer finishes, validate the index:
  - Document count is greater than 0.
  - A test query in Search Explorer returns relevant content from your documents.

---

### 4. Grounded Agent

Build the AI agent in Microsoft Foundry and connect it to your knowledge index.

- In your Microsoft Foundry project, create a new agent using the chat model you deployed.
- Add Azure AI Search as a knowledge source on the agent, pointing to your index.
- Write a system prompt that instructs the agent to:
  - Always ground answers in retrieved documents - no general knowledge for policy questions.
  - Include a source citation in every response.
  - Return a clear fallback message when the answer is not found in the index.
  - Format policy comparisons as a structured table.
- Test the agent in the Microsoft Foundry chat playground. Run at least five queries covering different scenarios - a direct lookup, a summary, a cross-document comparison, a recommendation, and an out-of-scope question. Confirm every in-scope answer includes a citation.

---

## Success Criteria

Before moving to Challenge 2, confirm the following:

- All 10 documents are uploaded to the `knowledge-docs` blob container.
- Both models (chat and embedding) are deployed in your Microsoft Foundry project.
- The AI Search index exists and document count is greater than 0.
- A test query in Search Explorer returns relevant results.
- The Microsoft Foundry agent returns cited, grounded responses in the playground for at least 5 test queries.
- The agent returns a clear fallback message (not a hallucinated answer) for an out-of-scope question.

---

Click **Next** at the bottom of the page to proceed to Challenge 2.