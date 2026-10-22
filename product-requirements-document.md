# Product Requirements Document (PRD)

**Title:** Blocked Supplier Invoice Analyser
**Date:** 2026-10-19
**Solution Category:** n8n Workflow, MCP Server, AI Agent

## Product Purpose & Value Proposition

**Elevator Pitch:**
When a supplier invoice is blocked in SAP, the accounts payable team needs to know why — quickly and clearly. This solution automates the lookup of all relevant data (invoice details, purchase order, goods receipt, vendor) and uses an AI agent to deliver a plain-language explanation of the block reason and a recommended corrective action, without the user having to navigate multiple SAP screens.

**Business Need:**
Blocked invoices create delays in the Source to Pay process. Today, resolving a blocked invoice requires manual navigation across SAP transactions to gather invoice details, cross-reference the purchase order and goods receipt, and check vendor data — before any diagnostic work can begin. This solution eliminates that manual effort by automating data retrieval and analysis.

**Expected Value:**
Faster blocked invoice resolution, reduced time spent on manual SAP navigation, and a consistent, clearly communicated outcome for every blocked invoice query.

**Product Objectives:**
1. Return a clear, plain-language explanation of the block reason for any blocked invoice.
2. Provide a specific recommended action alongside the explanation.
3. Handle missing or invalid invoice numbers gracefully with a helpful message.

## Requirements

### Must-Have Requirements

**R1: Accept Invoice Number via Webhook**
- **User Story:** As an accounts payable specialist, I need to trigger invoice analysis by providing an invoice number so that I can get an instant explanation without logging into SAP.
- **Acceptance Criteria:**
  - Given a valid invoice number is provided in the webhook request body, when the workflow is triggered, then processing begins immediately.
  - Given an empty or missing invoice number, when the request arrives, then a clear validation message is returned.
- **Priority Rank:** 1

**R2: Retrieve Blocked Invoice Details from SAP**
- **User Story:** As an accounts payable specialist, I need the workflow to fetch the invoice header and block status from SAP so that the analysis is based on live data.
- **Acceptance Criteria:**
  - Given a valid invoice number, when the MCP client queries SAP, then invoice header fields and the block reason code are returned.
  - Given the invoice number does not exist in SAP, when the query returns no result, then a helpful not-found message is returned to the caller — not a system error.
- **Priority Rank:** 2

**R3: Retrieve Related Purchase Order and Goods Receipt**
- **User Story:** As an accounts payable specialist, I need the related purchase order and goods receipt to be fetched automatically so that I do not have to look them up manually.
- **Acceptance Criteria:**
  - Given an invoice with a linked PO, when the workflow fetches PO data, then PO line items and quantities are available for analysis.
  - Given an invoice with a related goods receipt, when the workflow fetches GR data, then received quantities and dates are available for analysis.
- **Priority Rank:** 3

**R4: Retrieve Vendor Details**
- **User Story:** As an accounts payable specialist, I need the vendor's details to be included so that the analysis can consider vendor-specific factors in the block reason.
- **Acceptance Criteria:**
  - Given a valid vendor number on the invoice, when the workflow fetches business partner data, then vendor name, payment terms, and blocking status are available.
- **Priority Rank:** 4

**R5: AI Agent Analyses Block Reason**
- **User Story:** As an accounts payable specialist, I need an intelligent analysis of why the invoice is blocked so that I understand the root cause without expert SAP knowledge.
- **Acceptance Criteria:**
  - Given all SAP data has been retrieved, when the AI agent processes the context, then it produces a plain-language explanation of the block reason.
  - The explanation references specific data points (e.g., quantity mismatch between invoice and GR, price variance vs. PO).
- **Priority Rank:** 5

**R6: Return Plain-Language Explanation and Recommended Action**
- **User Story:** As an accounts payable specialist, I need a clear recommended action alongside the explanation so that I know exactly what to do next.
- **Acceptance Criteria:**
  - Given a completed AI analysis, when the workflow responds to the webhook, then the response contains both a block reason explanation and a recommended next step.
  - The language is plain and free of SAP technical codes or jargon.
- **Priority Rank:** 6

**R7: Read-Only Operation**
- **User Story:** As a finance manager, I need to be confident that this workflow never modifies any data in SAP so that there is no risk of unintended changes.
- **Acceptance Criteria:**
  - The workflow and MCP server only use read operations against SAP APIs.
  - No create, update, or delete calls are issued under any circumstance.
- **Priority Rank:** 7

## Solution Architecture

**Architecture Overview:**
A webhook-triggered n8n workflow receives an invoice number, reads all related SAP data via a new MCP Server, passes the assembled context to an AI agent for analysis, and returns a structured plain-language response to the caller.

**Key Components:**
- **n8n Workflow:** Orchestrates the end-to-end flow — trigger, SAP data retrieval, AI analysis, and response.
- **SAP MCP Server:** Exposes read-only tools for the four required SAP APIs (Supplier Invoice, Purchase Order, Goods Receipt Confirmations, Business Partner).
- **AI Agent (CUSTOM.sapAgent):** Receives the assembled SAP context and produces the block reason explanation and recommended action.

**Integration Points:**
- `API_SUPPLIERINVOICE_PROCESS_SRV` — read invoice header and block status
- `CE_PURCHASEORDER_0001` — read purchase order details
- `SSPOPENITMGOODSRECEIPT_0001` — read goods receipt confirmations
- `API_BUSINESS_PARTNER` — read vendor / business partner details

### Automation & Agent Behaviour

**Automation Level:** AI Agent-assisted, fully automated for data retrieval; open-ended reasoning for block analysis.

**Actions the system performs without human approval:**
- Fetching invoice, PO, GR, and vendor data from SAP
- Analysing the block reason and generating a recommendation

**Actions that require human review or approval:**
- None — this workflow is advisory only. All corrective actions remain with the human.

**Model or engine used:** SAP Generative AI Hub (via CUSTOM.sapAgent / CUSTOM.sapOrchestrationChatModel)

**Knowledge & data sources accessed:**
- SAP S/4HANA: invoice, purchase order, goods receipt, and vendor master data (read-only)

**Tools or connectors invoked:**
- Supplier Invoice API (read-only): retrieve invoice header and block reason code
- Purchase Order API (read-only): retrieve linked PO line items and quantities
- Goods Receipt API (read-only): retrieve GR confirmations and received quantities
- Business Partner API (read-only): retrieve vendor name, payment terms, and blocking status

**Guardrails & fail-safes:**
- The workflow and MCP server are strictly read-only — no write, update, or delete operations are permitted.
- If the invoice is not found, the workflow returns a user-friendly message and does not propagate a system error.
- If the AI agent cannot produce a confident analysis, it returns a clear statement that a manual review is needed.
- If any SAP data call fails, the workflow returns a graceful error message to the caller.
