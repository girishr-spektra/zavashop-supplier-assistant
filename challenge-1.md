# Challenge 01: Build the Supplier Review Assistant

## Overview

ZavaShop's procurement team manually reviews supplier quotes - checking prices against catalog benchmarks, pulling up supplier history, and writing a recommendation. The process takes 45 minutes per document.

In this challenge you will build an AI-powered supplier review assistant from scratch. You will provision the infrastructure, load the reference data into a searchable knowledge base, create a Foundry agent, and then write a system prompt that turns a generic assistant into a procurement specialist that produces structured, grounded review outputs.

---

## Prerequisites

Confirm you have the three data files downloaded from Getting Started before you begin:
- `supplier-quote.md`
- `product-catalog.csv`
- `supplier-history.md`

---

## Objectives

### 1. Set Up the Data Foundation

Create an Azure Storage account and upload the reference data files that will feed the knowledge base.

- In the Azure Portal, search for **Storage accounts** and create a new storage account in your assigned resource group. Use the following settings:
  - Performance: **Standard**
  - Redundancy: **LRS** (Locally Redundant Storage)
  - Leave all other settings at their defaults.
- Once the storage account is created, navigate to **Containers** and create a new container named `supplier-docs`. Set the access level to **Private**.
- Upload all three data files to the container:
  - `supplier-quote.md`
  - `product-catalog.csv`
  - `supplier-history.md`
- Confirm all three files appear in the container before proceeding.

---

### 2. Create a Microsoft Foundry Project and Deploy Models

Set up the AI environment that will power the agent and the vector search.

- Navigate to [ai.azure.com](https://ai.azure.com) and sign in with your challenge credentials.
- Create a new **Microsoft Foundry Hub** in your resource group (or use the pre-provisioned hub if one exists in your environment). Inside the hub, create a new **Project**.
- In your project, navigate to **Models and Endpoints** and deploy the following two models:

  | Deployment name | Model | Purpose |
  |---|---|---|
  | `gpt-4o-mini` | `gpt-4.1-mini` (or `gpt-4.1`) | Chat - powers the agent |
  | `text-embedding-ada-002` | `text-embedding-ada-002` | Embeddings - powers vector search |

- Wait for both deployments to show status **Succeeded** before continuing.

  > **Note:** If `gpt-4.1-mini` is unavailable in your region, use `gpt-4o-mini` as an alternative. Either model is sufficient for this challenge.

---

### 3. Build the Knowledge Base with Azure AI Search

Create an Azure AI Search service and import the reference documents as a vectorized index.

- In the Azure Portal, search for **AI Search** and create a new service in your resource group. Select the **Basic** pricing tier. Leave all other settings at their defaults.
- Once the service is created, navigate to it and select **Import and vectorize data**.
- On the **Connect your data** step, select **Azure Blob Storage** as the data source. Point it to the `supplier-docs` container you created in Objective 1.
- On the **Vectorize your text** step, select your Microsoft Foundry project and choose the `text-embedding-ada-002` deployment as the embedding model.
- Complete the import wizard and wait for the indexer run to finish. Confirm:
  - Index document count is greater than 0.
  - No errors appear in the indexer run history.

  > **Note:** The indexer processes all three files in the container. After the run completes, the index will contain chunks from the supplier quote, product catalog, and supplier history documents.

---

### 4. Create the Agent and Connect the Knowledge Source

Build the Foundry agent, connect the AI Search index, and test it with structured procurement prompts.

- In your Foundry project, navigate to **Agents** and create a new agent. Give it a descriptive name such as `ZavaShop Supplier Review Agent`.
- Assign your `gpt-4o-mini` (or `gpt-4.1`) deployment as the model.
- Under **Knowledge**, add a new knowledge source. Select **Azure AI Search** and connect it to the index you created in Objective 3.
- In the **Instructions** (system prompt) field, enter the following:

  ```
  You are a procurement review assistant for ZavaShop.

  When given a supplier quote or document, do the following:

  1. Extract the key commercial terms: Supplier Name, Quoted Items, Unit Prices, Minimum Order Quantities, Payment Terms, Delivery Timeline, Contract Duration, Renewal Terms, and any Exceptions. For any field that is missing or ambiguous in the document, flag it clearly as [MISSING] or [UNCLEAR].

  2. Search your knowledge base for relevant product catalog benchmarks and supplier history for the supplier and SKUs mentioned.

  3. Produce a Procurement Review Summary in this exact format:
     - Recommendation: Approve / Approve with Conditions / Escalate for Review (one sentence basis)
     - Supporting Reasons: 2-4 bullets grounded in the document and knowledge base
     - Risks: list each risk and cite the source (catalog policy, supplier history, or document clause)
     - Open Questions: specific questions to raise with the supplier before proceeding

  Only use information from the document and your knowledge base. Do not make up data.
  ```

- Save the agent and open the **Playground**.
- Test the agent with the following prompts in order:

  **Prompt 1 - Greet and orient:**
  ```
  What are you designed to help with?
  ```

  **Prompt 2 - Structured extraction:**
  ```
  Extract the commercial terms from this supplier quote and flag anything missing or unclear.

  [paste the full contents of supplier-quote.md here]
  ```

  **Prompt 3 - Full procurement review:**
  ```
  Now produce a full procurement review summary for this quote. Check the catalog benchmarks and supplier history before forming your recommendation.
  ```

- Evaluate the Prompt 3 output against these quality checks:
  - Recommendation is one of the three valid options (Approve / Approve with Conditions / Escalate for Review).
  - At least one risk references a specific figure or policy from the product catalog.
  - At least one risk or open question references the supplier history data.
  - Missing or ambiguous fields from the document are surfaced in Open Questions.

---

## Success Criteria

Before completing the challenge, confirm all of the following:

- Azure Storage account and `supplier-docs` container created with all three files uploaded.
- Microsoft Foundry project exists with both model deployments showing status Succeeded.
- AI Search index is populated (document count greater than 0, no indexer errors).
- Foundry agent is connected to the AI Search knowledge source.
- Prompt 2 returns a structured extraction with at least one field flagged as `[MISSING]` or `[UNCLEAR]`.
- Prompt 3 returns a Procurement Review Summary with Recommendation, Supporting Reasons, Risks, and Open Questions.
- Risks and reasons in the output are traceable to the source data - not generic statements.

---

## Congratulations

You have built a fully grounded procurement review assistant. The infrastructure pattern you followed - blob storage as the data foundation, AI Search for retrieval, Microsoft Foundry as the reasoning layer - is the same pattern used in production-grade RAG applications.

If you have time and want to go further, take on the bonus challenge below.

---

## Bonus Challenge: Multi-Agent Procurement Pipeline

### Why Multi-Agent?

The single agent you built works well, but it is doing two very different jobs in one system prompt - extraction and recommendation. In production, that creates problems: the extraction logic and the reasoning logic are hard to test independently, errors in one phase pollute the other, and you cannot reuse either specialist elsewhere.

The fix is to split responsibilities across two specialist agents and wire them together through an orchestrator. This is the multi-agent pattern and it is how real enterprise AI systems are built.

**What you will build:**

```
User message (supplier quote)
        |
        v
  Orchestrator Agent
  (routes, sequences, assembles)
        |
        |---> Extraction Agent
        |     (reads raw document, returns structured fields + flags)
        |
        |---> Analyst Agent
              (receives structured extraction, queries knowledge base, produces review)
```

---

### Step 1 - Create the Extraction Specialist Agent

- In your Foundry project, create a second agent named `ZavaShop Extraction Agent`.
- Assign the same chat model deployment.
- Do NOT connect any knowledge source to this agent. It only reads the document it is given.
- Write a tight, single-purpose system prompt:

  ```
  You are a document extraction specialist for ZavaShop procurement.

  When given a supplier document, extract ONLY the following fields and return them in a structured list:

  - Supplier Name
  - Quoted Items (one line per SKU: name, SKU code, unit price, MOQ)
  - Payment Terms
  - Delivery Timeline
  - Contract Duration
  - Renewal Terms
  - Exceptions and Exclusions

  Rules:
  - If a field is missing from the document, output: [MISSING]
  - If a field is present but ambiguous or contradictory, output: [UNCLEAR - brief reason]
  - Do not interpret, infer, or recommend. Only extract what is explicitly stated.
  - Do not output anything except the structured field list.
  ```

- Test this agent alone in its own playground session by pasting the contents of `supplier-quote.md`. Confirm:
  - Output is structured and consistent.
  - Delivery Timeline is flagged `[MISSING]` (it is not confirmed in the document).
  - Renewal Terms are flagged `[UNCLEAR]` (no fixed notice period, no capped price escalation).
  - No recommendations or opinions appear in the output.

---

### Step 2 - Create the Analyst Specialist Agent

- Create a third agent named `ZavaShop Analyst Agent`.
- Assign the same chat model deployment.
- Connect the AI Search knowledge source to this agent (same as Objective 4).
- Write a system prompt focused only on analysis - assume it always receives pre-extracted structured data as input:

  ```
  You are a procurement analyst for ZavaShop.

  You will receive structured commercial data that has already been extracted from a supplier document. Your job is to analyse it and produce a procurement review.

  Steps:
  1. Search the knowledge base for catalog benchmarks and supplier history matching the supplier name and SKU codes in the input.
  2. Compare each quoted price against the catalog benchmark. Flag any price more than 5% above benchmark.
  3. Check supplier history for delivery performance, quality incidents, and contract compliance issues.
  4. Produce a Procurement Review Summary:
     - Recommendation: Approve / Approve with Conditions / Escalate for Review (cite your primary reason)
     - Supporting Reasons: 2-4 bullets, each grounded in the data
     - Risks: each risk on a separate line, cite the source (catalog policy, supplier history entry, or document clause)
     - Open Questions: questions to raise with the supplier for any [MISSING] or [UNCLEAR] fields

  Base every statement on the structured input and knowledge base only. Do not fabricate data.
  ```

- Test this agent by pasting the structured output from your Extraction Agent (from Step 1) as the input message. Confirm:
  - The analyst references specific prices and compares them against catalog benchmarks.
  - At least one risk cites the supplier history.
  - Open Questions address the flagged fields.

---

### Step 3 - Wire Them Together with an Orchestrator

- Create a fourth agent named `ZavaShop Procurement Orchestrator`.
- Assign the same chat model deployment.
- In the **Connected Agents** section (or **Tools**), add both specialist agents as callable agents:
  - `ZavaShop Extraction Agent`
  - `ZavaShop Analyst Agent`
- Write the orchestrator system prompt:

  ```
  You are the procurement orchestration agent for ZavaShop.

  When a user submits a supplier document or quote:
  1. Call the Extraction Agent with the full document content. Wait for its structured output.
  2. Pass the structured output to the Analyst Agent. Wait for its procurement review.
  3. Return both outputs to the user: first the structured extraction, then the full procurement review below it.

  Do not perform extraction or analysis yourself. Delegate to the specialist agents and assemble the result.
  ```

- Open the Orchestrator's playground and paste the full contents of `supplier-quote.md` as your message.
- Confirm the final output contains both sections:
  - Structured extraction with flags from the Extraction Agent.
  - Procurement Review Summary with recommendation and grounded risks from the Analyst Agent.

---

### Bonus Success Criteria

- Three agents exist: Extraction Specialist, Analyst Specialist, Orchestrator.
- Extraction Agent returns structured output with no opinions or recommendations.
- Analyst Agent returns a review grounded in the knowledge base.
- Orchestrator produces a combined output from a single user message containing the raw document.
- You can explain to someone else why splitting these responsibilities across agents is better than a single agent for a production system.

---

## Congratulations

You have built a multi-agent procurement pipeline. The orchestrator-specialist pattern you used here scales to real enterprise scenarios: swap the static files for a live SharePoint or ERP feed, add a third agent that routes the review to the right approver based on recommendation type, or expose the orchestrator through Copilot Studio so procurement staff can trigger a review from Teams by pasting a quote.

The architecture you built today is production-grade.
