# Blocked Supplier Invoice Analyser

Automated workflow that retrieves all relevant data for a blocked supplier invoice from SAP and uses an AI agent to explain the block reason and recommend a corrective action — in plain language.

## Business Challenge

When a supplier invoice is blocked in SAP, the accounts payable team needs to quickly understand why it is blocked — including the related purchase order, goods receipt, and vendor details — and know what action to take. Currently this requires manual navigation across multiple SAP transactions. The solution must be read-only, return a clear plain-language explanation, and handle missing invoices gracefully with a helpful message rather than a system error.

## Business Architecture (RBA)

### End-to-End Process

Source to Pay (generic)

### Process Hierarchy

```
Source to Pay (E2E)
└── Procure to Receipt (generic)
    └── Purchase products and services (generic) [BPS-329]
        └── Manage purchase order
        └── Verify supplier invoice
        └── Resolve invoice block
```

### Summary

Blocked invoice resolution sits within the Source to Pay E2E process, spanning invoice verification, purchase order management, and goods receipt confirmation — with SAP S/4HANA as the system of record.

## Fit Gap Analysis

| Requirement (business) | Standard asset(s) found | API ORD ID | MCP Server ORD ID | MCP Server Version | Webhook API ORD ID | Gap? | Notes / assumptions |
| ---------------------- | ----------------------- | ---------- | ----------------- | ------------------ | ------------------ | ---- | ------------------- |
| Read blocked supplier invoice details | SAP S/4HANA | `sap.s4:apiResource:API_SUPPLIERINVOICE_PROCESS_SRV:v1` | — | — | — | No | API available in landscape; new MCP server asset required |
| Read related purchase order | SAP S/4HANA | `sap.s4:apiResource:CE_PURCHASEORDER_0001:v1` | — | — | — | No | API available in landscape; new MCP server asset required |
| Read goods receipt confirmation | SAP S/4HANA | `sap.s4:apiResource:SSPOPENITMGOODSRECEIPT_0001:v1` | — | — | — | No | API available in landscape; new MCP server asset required |
| Read vendor / business partner details | SAP S/4HANA | `sap.s4:apiResource:API_BUSINESS_PARTNER:v1` | — | — | — | No | API available in landscape; new MCP server asset required |
| AI analysis of block reason + plain-language recommendation | Custom AI Agent | — | — | — | — | No | CUSTOM.sapAgent node available; new AI Agent asset required |
| Webhook trigger (caller initiates analysis) | n8n | — | — | — | — | No | `n8n-nodes-base.webhook` + `n8n-nodes-base.respondToWebhook` available |
| Graceful not-found handling | n8n | — | — | — | — | No | `n8n-nodes-base.if` node used to branch on missing invoice |

### Key findings

- Four SAP APIs are available in the landscape: Supplier Invoice, Purchase Order, Goods Receipt, and Business Partner.
- No pre-built MCP servers exist for these APIs; a new MCP Server asset must be created to expose them as tools.
- The CUSTOM.sapAgent and CUSTOM.sapMcpClient nodes are available on this n8n instance — the AI analysis pattern is fully supported.
- The solution is strictly read-only; no write-back to SAP is required or planned.
- Invoice not-found scenarios must return a user-friendly message via the Respond to Webhook node, not a system error.
- Several OP_ (on-premise) API variants are marked unavailable; the cloud-edition equivalents are used instead.

## Recommendations

### Blocked Invoice Intelligent Analyser

#### Executive Summary

n8n workflow + MCP Server + AI Agent for read-only blocked invoice analysis

#### Recommended Solution

A webhook-triggered n8n workflow collects the invoice number from the caller, reads all related SAP data (invoice header and block status, linked purchase order, goods receipt, and vendor details) via a new SAP MCP Server, then passes the assembled context to an AI Agent. The agent analyses the data and returns a plain-language explanation of the block reason and a recommended action. If the invoice is not found, the workflow returns a helpful message. No changes are made in SAP.

#### Recommended solution category

n8n Workflow, MCP Server, AI Agent

#### Intent fit
95%
