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

1. In the Azure Portal, search for **Storage accounts** and create a new storage account in your assigned resource group. Use the following settings:

- Performance: **Standard**
- Redundancy: **LRS** (Locally Redundant Storage)
- Leave all other settings at their defaults.

2. Once the storage account is created, navigate to **Containers** and create a new container named `supplier-docs`. Set the access level to **Private**.

3. Upload all three data files to the container:

- `supplier-quote.md`
- `product-catalog.csv`
- `supplier-history.md`

4. Confirm all three files appear in the container before proceeding.

<validation step="c93fcf6d-d226-4bf2-b04b-5329ce269d57" />
 
> **Congratulations** on completing the Challenge! Now, it's time to validate it. Here are the steps:
> - Hit the Validate button for the corresponding Challenge. If you receive a success message, you can proceed to the next Challenge. 
> - If not, carefully read the error message and retry the step, following the instructions in the lab guide.
> - If you need any assistance, please contact us at cloudlabs-support@spektrasystems.com. We are available 24/7 to help.

---

### 2. Create a Microsoft Foundry Project and Deploy Models

Set up the AI environment that will power the agent and the vector search.

- Create a new **Microsoft Foundry Project** directly in your resource group. You do not need to create a hub first. Once deployed, navigate to Microsoft Foundry project.
- In your project, navigate to **Models and Endpoints** and deploy the following two models:

  | Deployment name | Model | Purpose |
  |---|---|---|
  | `gpt` | `gpt-4.1-mini` (or `gpt-4.1`) | Chat - powers the agent |
  | `text-embedding-ada-002` | `text-embedding-ada-002` | Embeddings - powers vector search |

- Wait for both deployments to show status **Succeeded** before continuing.

  > **Note:** If `gpt-4.1-mini` is unavailable in your region, use an alternative region.

<validation step="cabc6d3d-c876-4132-96b0-ee4601f246c9" />
 
> **Congratulations** on completing the Challenge! Now, it's time to validate it. Here are the steps:
> - Hit the Validate button for the corresponding Challenge. If you receive a success message, you can proceed to the next Challenge. 
> - If not, carefully read the error message and retry the step, following the instructions in the lab guide.
> - If you need any assistance, please contact us at cloudlabs-support@spektrasystems.com. We are available 24/7 to help.

---

### 3. Build the Knowledge Base with Azure AI Search

Create an Azure AI Search service and import the reference documents as a vectorized index.

- In the Azure Portal, search for **AI Search** and create a new service in your resource group. Select the **Basic** pricing tie (if not available as an alternative you can choose standard. Leave all other settings at their defaults.
- Once the service is created, navigate to it and select **Import and vectorize data**.
- On the **Connect your data** step, select **Azure Blob Storage** as the data source, then select the **RAG** option to set up a retrieval-augmented generation index.
- Point the data source to the `supplier-docs` container you created in Objective 1.
- On the **Vectorize your text** step, select your Microsoft Foundry project and choose the `text-embedding-ada-002` deployment as the embedding model.
- Complete the import wizard and wait for the indexer run to finish. Confirm:
  - Index document count is greater than 0.
  - No errors appear in the indexer run history.

  > **Note:** The indexer processes all three files in the container. After the run completes, the index will contain chunks from the supplier quote, product catalog, and supplier history documents.

<validation step="38f42e75-2e8d-49d2-affd-65fe33949ee3" />
 
> **Congratulations** on completing the Challenge! Now, it's time to validate it. Here are the steps:
> - Hit the Validate button for the corresponding Challenge. If you receive a success message, you can proceed to the next Challenge. 
> - If not, carefully read the error message and retry the step, following the instructions in the lab guide.
> - If you need any assistance, please contact us at cloudlabs-support@spektrasystems.com. We are available 24/7 to help.

---

### 4. Create the Agent and Connect the Knowledge Source

Build the Foundry agent, connect the AI Search index, and test it with structured procurement prompts.

- In your Foundry project, navigate to **Agents** and select **+ New agent**.
- Give the agent a descriptive name such as `ZavaShop-Review-Agent`.
- Assign your `gpt-4.1-mini` (or `gpt-4.1`) deployment as the model.
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

Click **Next** at the bottom of the page to proceed to Bonus Challenge.

---
