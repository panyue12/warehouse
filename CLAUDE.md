# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Single-file warehouse inventory management system (`warehouse.html`). No build process, no framework — open the file directly in a browser. All data persists in `localStorage` under key `warehouse_system_data`. The only external dependency is SheetJS (xlsx) loaded from CDN for Excel import/export.

## Data model (in localStorage JSON)

| Field | Description |
|---|---|
| `materials[]` | `{code, name, unit, stock}` — 配料 (raw materials/parts) |
| `products[]` | `{code, name, unit, stock}` — 成品 (finished products) |
| `bom[]` | `{productCode, materialCode, quantity}` — how much of each material a product consumes on production |
| `productIn[]` | `{time, productCode, productName, quantity, operator, note}` |
| `productOut[]` | `{time, productCode, productName, quantity, operator, note}` |
| `materialIn[]` | `{time, materialCode, materialName, quantity, operator, note}` |
| `materialOut[]` | `{time, materialCode, materialName, quantity, operator, note, type}` — `type` is `"auto"` or `"manual"` |
| `logs[]` | `{time, msg, snapshot?}` — recent history; snapshots stored only for the last 20 entries |

## Key business logic (productInbound)

When a finished product is "inbound" (produced), the system:
1. Looks up all BOM entries for that product
2. Checks each material has sufficient stock (`mat.stock >= bomQty × productionQty`)
3. Deducts materials and creates `materialOut` records with `type: "auto"`
4. Increments the product's stock
5. Creates a `productIn` record

This is the core transaction — material consumption is automatic on production. Material outbound via the manual tab is for losses/adjustments.

## Entry points

- Open `warehouse.html` in a browser — no server needed
- No tests, no linting, no build step
- `esc()` function on line 1109 handles XSS prevention for all rendered user data
- `MAX_SNAPSHOTS = 20` on line 296 controls how many log entries retain full data snapshots for rollback
