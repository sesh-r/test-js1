# Specification

> **Guidelines**: Read [guidelines.md](./guidelines.md) before executing ANY tasks below.

Check off items as completed.

## Solution Setup

- [x] Create asset directories:
  ```
  mkdir -p assets/workflows/blocked-invoice-analyser-workflow
  mkdir -p assets/blocked-invoice-analyser-mcp
  ```
- [x] Invoke `setup-solution` skill to create `solution.yaml` and `asset.yaml` files for every asset
- [x] Validate all `asset.yaml` and `solution.yaml` files exist and are well-formed

## MCP Translation Files

- [x] Invoke `mcp-translation-file` skill for `specification/blocked-invoice-analyser-workflow/api-specs/supplier-invoice.edmx` (API ORD ID: `sap.s4:apiResource:API_SUPPLIERINVOICE_PROCESS_SRV:v1`, type: edmx)
- [x] Invoke `mcp-translation-file` skill for `specification/blocked-invoice-analyser-workflow/api-specs/purchase-order.edmx` (API ORD ID: `sap.s4:apiResource:CE_PURCHASEORDER_0001:v1`, type: edmx)
- [x] Invoke `mcp-translation-file` skill for `specification/blocked-invoice-analyser-workflow/api-specs/goods-receipt.edmx` (API ORD ID: `sap.s4:apiResource:SSPOPENITMGOODSRECEIPT_0001:v1`, type: edmx)
- [x] Invoke `mcp-translation-file` skill for `specification/blocked-invoice-analyser-workflow/api-specs/business-partner.edmx` (API ORD ID: `sap.s4:apiResource:API_BUSINESS_PARTNER:v1`, type: edmx)

## Asset Implementation

- [x] Execute specification/blocked-invoice-analyser-workflow/specification.md (all items)
