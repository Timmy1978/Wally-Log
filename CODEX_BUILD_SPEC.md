# Wally-Log — Codex Build Specification

## Purpose
Build a dealership sales-log importer and month-end reporting application. This specification is the authoritative starting point for implementation. Validate source-column mappings against a real sample before trusting calculations.

## Source and import
- Import CSV/XLSX sales logs, support repeated monthly imports, and retain source row, import batch, date, stock number, deal identifiers, both salesperson fields, sale type, O/S classification, budget designation, front gross, F&I, trade, GAP, warranty, vehicle age, and lead source where available.
- Preserve original raw values, normalize labels and salesperson aliases, and flag missing/ambiguous codes. Never silently treat unknown codes as valid sales.
- Use stable transaction identifiers and duplicate detection to prevent double-counting on re-import. Keep corrections auditable.

## Official unit counting
- Count each qualifying deal once at dealership level.
- Count BF (Buckeye Financial) toward official salesperson unit totals.
- Exclude TLBO (true lease buyout) and NC (non-counter) from official salesperson unit totals, while displaying their transaction counts and financial/activity metrics separately.
- Distinguish New, Used, and BF as applicable; keep BF gross and F&I separate from conventional retail gross and F&I.
- O = Optional, S = Standard, Bud = Budget. Display O, S, Bud, BF, TLBO and NC per salesperson. These may be overlapping dimensions; do NOT sum O+S+Bud+BF as though mutually exclusive unless confirmed by source data. Preserve each transaction's tags.
- Preserve any other existing exclusions from retail financial metrics, and confirm classification mappings from source log before implementation.

## Split deals
- When two different names appear in the salesperson columns, allocate exactly 50% to each. When only one name appears, allocate 100%. Do not create a second allocation when the same person appears twice.
- Apply the same allocation factor to eligible unit credit (0.5 each), O/S/Bud/BF/TLBO/NC activity, front gross, F&I gross, total gross, trade credit, GAP, warranty, and any other additive transaction metric.
- At dealership level retain original transaction totals, not doubled allocations. For per-salesperson TLBO/NC activity show fractional credit on split deals, with separate transaction-level counts available in detail.
- Apply the TLBO/NC exclusion to official units *before* allocation.
- Nonadditive measures (percentages, PVR, average vehicle age) must be recomputed from weighted numerators/denominators rather than simply halved.

## Reports
1. Dealership monthly dashboard: qualifying new/used/BF units, O/S/Bud mix, TLBO and NC excluded activity, conventional retail front/F&I/total gross, separate BF production, trade counts/rate, product counts/penetration, aged inventory, lead source and month-over-month comparison.
2. Salesperson table: EVERY distinct name in either salesperson column, including managers/BDC/one-off names; columns for O, S, Bud, BF, TLBO, NC, official units, new/used units, conventional front/F&I/total gross, front/F&I/total PVR, trades, trade rate, GAP, warranties, aged units. Allow sorting/filtering.
3. Click-through person detail: source transactions, original deal amounts, allocated shares, exclusion reasons, and subtotals.
4. Separate manager/dealership totals and exportable CSV/XLSX reports.

## Calculation and data integrity
- Official salesperson unit total = sum of allocated qualifying unit credits (BF included; TLBO and NC excluded).
- Front/F&I/total retail PVR = allocated eligible conventional retail gross / allocated eligible conventional retail units, excluding BF/TLBO/NC unless explicit approved accounting rules specify otherwise.
- Trade rate and product penetration must use clearly labeled eligible denominators. BF must not inflate conventional retail F&I, trade, or product metrics.
- Sum of person-level allocations must reconcile to each included deal's original totals; report mismatches, missing names, invalid split inputs, duplicates and unmapped categories.
- Distinguish financial reporting of excluded transactions from official sales counts; show TLBO/NC financials separately so they do not silently inflate official retail metrics.
- Show report month and as-of date; allow regenerating reports from corrected imports.
- Add automated tests for solo, split, BF, TLBO, NC, BF+split, TLBO+split, repeated salesperson, missing salesperson, duplicate import, and overlapping O/S/Bud tags.

## Implementation approach
- First inspect actual uploaded sample logs and ask for missing code definitions; never invent column mappings.
- Implement data normalization, classification and allocation as testable pure functions before dashboard UI.
- Provide a simple import workflow, month selector, dashboard, salesperson detail, validation warnings and exports.
- No customer phone numbers or other personal information in test fixtures, screenshots, public repository commits, or logs.
