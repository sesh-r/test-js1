# Asset Specification: blocked-invoice-analyser-workflow

> **Guidelines**: Read [../guidelines.md](../guidelines.md) and [../guidelines-n8n-workflow.md](../guidelines-n8n-workflow.md) before executing any tasks below.

**Asset type:** n8n Workflow  
**Asset folder:** `assets/workflows/blocked-invoice-analyser-workflow/`  
**Workflow file:** `blocked-invoice-analyser.n8n.json`

## Data Dependencies

### MCP API: sap.s4:apiResource:API_SUPPLIERINVOICE_PROCESS_SRV:v1
MCP tool scope: read-only  
Resources in scope: A_SupplierInvoice, A_SupplierInvoiceItem

### MCP API: sap.s4:apiResource:CE_PURCHASEORDER_0001:v1
MCP tool scope: read-only  
Resources in scope: PurchaseOrder, PurchaseOrderItem

### MCP API: sap.s4:apiResource:SSPOPENITMGOODSRECEIPT_0001:v1
MCP tool scope: read-only  
Resources in scope: GoodsReceiptConfirmation

### MCP API: sap.s4:apiResource:API_BUSINESS_PARTNER:v1
MCP tool scope: read-only  
Resources in scope: A_BusinessPartner, A_BusinessPartnerAddress

API spec files are at:
- `specification/blocked-invoice-analyser-workflow/api-specs/supplier-invoice.edmx`
- `specification/blocked-invoice-analyser-workflow/api-specs/purchase-order.edmx`
- `specification/blocked-invoice-analyser-workflow/api-specs/goods-receipt.edmx`
- `specification/blocked-invoice-analyser-workflow/api-specs/business-partner.edmx`

## MCP Server Setup

- [ ] Invoke the `mcp-translation-file` skill to generate translation.json files from each of the four API spec files above
- [ ] Create an MCP server asset for the four translation files under `assets/blocked-invoice-analyser-mcp/`
- [ ] Reference the MCP server in the workflow asset.yaml `requires` block

## Workflow Setup

- [ ] Invoke the `setup-solution` skill to scaffold the workflow asset folder `assets/workflows/blocked-invoice-analyser-workflow/` with a valid `asset.yaml` (type: `n8nworkflow`) and a `workflow.ordId` placeholder
- [ ] Run the `generate-workflow-asset.js` script to populate `provides.apis[]` and `workflow.ordId` in `asset.yaml`
- [ ] Ensure `sourceRoot: "."` in `asset.yaml`

## Workflow Implementation

- [ ] Create `assets/workflows/blocked-invoice-analyser-workflow/blocked-invoice-analyser.n8n.json`

### Webhook Trigger
- [ ] Add a **Webhook** node (`n8n-nodes-base.webhook`) as the entry point
  - Method: POST
  - Path: `/analyse-blocked-invoice`
  - Response mode: `responseNode` (response is sent by the Respond to Webhook node at the end)
  - Accept JSON body with field: `invoiceNumber` (string)

### Input Validation
- [ ] Add an **IF** node (`n8n-nodes-base.if`) to check that `invoiceNumber` is present and non-empty
  - True branch → proceed to SAP data retrieval
  - False branch → route to a **Respond to Webhook** node returning a 400 message: "Please provide an invoiceNumber in the request body."

### SAP Data Retrieval — Supplier Invoice
- [ ] Add a **SAP MCP Client** node (`CUSTOM.sapMcpClient`) to fetch the invoice header and block status
  - Tool: read supplier invoice by `invoiceNumber`
  - Read-only; no write operations

### Not-Found Check
- [ ] Add an **IF** node to check whether the invoice was found in SAP
  - True branch (found) → continue
  - False branch → route to a **Respond to Webhook** node returning a 404 message: "Invoice [invoiceNumber] was not found in SAP. Please verify the invoice number and try again."

### SAP Data Retrieval — Purchase Order
- [ ] Add a **SAP MCP Client** node to fetch the related purchase order using the PO reference from the invoice

### SAP Data Retrieval — Goods Receipt
- [ ] Add a **SAP MCP Client** node to fetch goods receipt confirmations linked to the purchase order

### SAP Data Retrieval — Business Partner / Vendor
- [ ] Add a **SAP MCP Client** node to fetch vendor details using the vendor number from the invoice

### AI Agent Analysis
- [ ] Add a **SAP Agent** node (`CUSTOM.sapAgent`) to analyse the assembled context
  - System prompt must instruct the agent to:
    - Analyse why the invoice is blocked based on the provided invoice data, PO, GR, and vendor details
    - Return a plain-language explanation of the block reason (no SAP codes or jargon)
    - Return a clear, specific recommended action for the AP specialist
    - If data is insufficient for a confident diagnosis, state that a manual review is required
    - Never suggest or imply any write-back or change in SAP
  - Pass all four SAP data payloads as context to the agent

### Response
- [ ] Add a **Respond to Webhook** node (`n8n-nodes-base.respondToWebhook`) as the final node
  - HTTP status: 200
  - Response body (JSON):
    ```json
    {
      "invoiceNumber": "...",
      "blockReasonExplanation": "...",
      "recommendedAction": "..."
    }
    ```

## Validation

- [ ] Run `validate-n8n-workflow` MCP tool against the workflow JSON and `asset.yaml`
- [ ] Fix all errors and warnings before marking complete
- [ ] Verify no credentials block is present in any node
- [ ] Verify all `connections` reference nodes by `name`, not `id`
- [ ] Verify no write/update/delete MCP tool calls exist anywhere in the workflow
