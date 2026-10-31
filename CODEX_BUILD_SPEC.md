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


## Gross profit and write-downs — Wally Summary
- Add New, Used, Wholesale, and Total rows; Wholesale is a planned future import category and must be supported without inventing historical records.
- For each row show distinct deal units, gross before write-downs, write-down amount, gross after write-downs, average gross/unit before write-downs, and average gross/unit after write-downs.
- Total includes Wholesale; sum each deal only once. Compute averages using distinct eligible transaction counts (not salesperson split credits). Zero-denominator averages display N/A.
- Gross after write-downs = gross before write-downs minus applicable write-downs; verify the sign convention and whether source gross is already net of write-downs, to prevent double deductions.
- Preserve front gross, F&I gross, and combined gross as separate accounting measures. Confirm whether the requested headline gross is front or combined before finalizing financial calculations.
- Determine classification for BF, Bud, TLBO and NC versus New/Used/Wholesale using the actual source data and approved rules; do not silently exclude or double count categories.
- Provide drilldown to source deals, write-down amounts, and any reconciliation errors.

## Alliance resident sales
- Add a salesperson-by-category Alliance customer report with columns Alliance New, Alliance Used, Alliance BF, Alliance Bud, and total distinct qualifying Alliance deals; prominently surface Alliance New separately.
- Match customer residence city Alliance, Ohio from the actual customer city/state fields, case-insensitively, with whitespace/abbreviation normalization; do not infer residence from dealership location. Flag missing/ambiguous city/state.
- Include every distinct salesperson name in either sales column, with 50/50 fractional attribution for splits and 100% for solo deals. Preserve distinct dealership transaction totals.
- BF and Bud may overlap New/Used; do not add overlapping columns to produce total. Calculate total from unique qualifying deals with salesperson allocations.
- Apply the existing TLBO/NC official-count exclusions to the Alliance qualifying totals, but permit separately visible excluded activity for audit.
- Show person detail with source stock/deal ID, sale type, city, classification, split credit and exclusion reason.

## Additional acceptance tests
- Validate new/used/wholesale rollup and averages before/after write-downs, including zero units, negative gross, write-down sign handling, and prevention of double deductions.
- Validate Alliance case-insensitive city matching, non-Alliance exclusion, BF/Bud overlap, 50/50 split, TLBO/NC exclusion, and dealership-versus-person reconciliation.
