## Goal
Prevent finished-product stock from going negative by validating at the point of issue/sale. Existing negative balances will surface naturally as users correct entries.

## Where negatives originate
Available stock is computed as: `produced (production_entries) − issued (stock_issues + finished-product sales)`. Negatives appear when:
1. A **Sale** of a finished product is recorded for more than what's available.
2. A **Stock Issue** (admin Stock Management) is recorded for more than what's available.

There is no validation in either form today.

## Changes

### 1. `src/pages/inventory/SalesEntry.tsx` (finished products)
- Before submit, when `tab === "finished_product"`, fetch current available stock for the selected `product_code_id`:
  - Sum `total_quantity` from `production_entries` for that product.
  - Subtract sum of `quantity` from `stock_issues` for that product.
  - Subtract sum of `quantity` from `sales` where `item_type='finished_product'` for that product.
- If `entered quantity > available`, show toast error ("Only X available") and abort insert.
- Also display "Available: X unit" inline under the product selector for visibility.

Raw-material sales already deduct via DB trigger from `raw_materials.current_stock`; add a parallel check there too — block sale if `quantity > raw_materials.current_stock`.

### 2. `src/pages/admin/StockManagement.tsx` (issue dialog)
- Available is already computed in `summaries`. Before inserting into `stock_issues`, look up the selected product's `available` and reject if `issueQuantity > available` with a toast.
- Keep the existing inline "Available" display.

### 3. Shared helper (optional, light)
Add a tiny utility `src/lib/stock.ts` with `getFinishedProductAvailable(productCodeId)` that runs the three queries above, so both forms use the same logic. Keeps drift between the two screens out.

## Out of scope (per user choice "Block over-issue at source")
- No display clamping in InventoryView — real numbers stay visible so admins can see existing discrepancies.
- No DB-level CHECK/trigger enforcement (would require backfill of current negatives first).
- No audit/report screen for currently-negative items.

## Validation
- Try to record a sale/issue larger than available → blocked with toast.
- Record one within available → succeeds; available decreases on the InventoryView refresh.
- Raw-material sale exceeding `current_stock` → blocked.