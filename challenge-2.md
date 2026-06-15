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

- In your Foundry project, select **+ New agent**, then **Create agent**, and select **Yes** when prompted with **Create legacy agent**. Name this second agent `ZavaShop-Extraction-Agent`.
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

- Select **+ New agent**, then **Create agent**, and select **Yes** when prompted with **Create legacy agent**. Name this third agent `ZavaShop-Analyst-Agent`.
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

- Select **+ New agent**, then **Create agent**, and select **Yes** when prompted with **Create legacy agent**. Name this fourth agent `ZavaShop-Procurement-Orchestrator`.
- Assign the same chat model deployment.
- In the **Connected Agents** section (or **Tools**), add both specialist agents as callable agents. As you add each one, Foundry asks for an **instruction** that tells the orchestrator when and how to call that agent. Add both connected agents with the instructions below.
  - Add `ZavaShop-Extraction-Agent` and set its instruction to:

    ```
    Call this agent first. Send it the full raw supplier document content. It returns the structured commercial fields with [MISSING] and [UNCLEAR] flags. Do not modify the document before sending it.
    ```

  - Add `ZavaShop-Analyst-Agent` and set its instruction to:

    ```
    Call this agent second. Send it the structured extraction returned by ZavaShop-Extraction-Agent. It queries the knowledge base and returns the Procurement Review Summary. Do not call it until the extraction output is available.
    ```

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
