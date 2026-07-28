# Lab Data Merging Program
**Final Documentation**
**Date:** July 15, 2026

---

## 1. Project Background

This is a one-off data merging program requested by management to reduce the manual effort required by Ms. Jam, a laboratory assistant in the color production department, when cross-referencing quality check (QC) results from two separate systems.

The laboratory uses a spectrophotometer unit called 3NH. This program merges the QC evaluation results with the 3NH Spectro readings into one final report.

The program covers five product codes: **BA2103E, RA16826E, RA18011E, WA12282E, WA15190E.**

---

## 2. The Two Source Systems

**3NH Spectro Export**
- One Excel file per product code.
- The first row (and sometimes the 2nd/3rd) is a reference/standard color row (STD, LIGHT, DARK) — never has a matching QC record.
- Contains a Judgement column (Pass/Fail) based on color difference only — not the QC evaluation result.
- Key column: Name, which holds the lot number (or, for certain clients, the internal lot number — see Matching Modes below).

**QC Evaluation System Export**
- CSV export covering all QC evaluation records, across all product codes.
- The raw export inserts one blank spacer row between every real record; this is dropped immediately after loading.
- Key column: lot_number (or internal_lot, depending on matching mode).

---

## 3. Matching Modes

- **Mode A (default):** Spectro's Name column holds the regular sticker lot number, matched directly against QC's lot_number.
- **Mode B (special):** Spectro's Name column holds the internal lot number instead, matched against QC's internal_lot (or a parenthesis-format fallback if internal_lot is blank). Applies only to specific clients: PACKAGEWORLD/ABI, ROWELL, DYNAMICCAPS/NICE — confirmed to cover 9 product codes: BA12556E, WA12282E, WA15151E, WA15816E, BA17042E, BA17070E, WA7997E, WA15218E, WA15229E.

---

## 4. Program Structure

### (A) QC File Validation
- Load and clean the QC file.
- Normalize text columns; check for missing/gapped IDs.
- Validate and correct product codes and lot numbers against confirmed formats (single lot, range, bag-in-parenthesis, internal-lot-in-parenthesis, typo corrections).
- Classify and expand lot number formats.
- Correct bag numbers that were mangled into date-like text by the QC export.
- Recover bag numbers from free-text remarks when the dedicated bag number field is blank — this is a confirmed, recurring data-entry pattern, not a one-off. Recognized formats: a single bag number, or a bag range. Recovery accounts for "collection bag" (megabag) references appearing alongside the true bag number in the same remark, so the correct number is captured rather than the collection reference.

### (B) Spectro Files Validation
- Load and normalize the Spectro files.
- Identify and separate reference rows (STD/LIGHT/DARK) from real lot data.
- Validate and correct the Name column format.
- Split Name into lot number and bag number, and flag oversize (OS) lots.
- Check bag number sequences for gaps and fill any gap confirmed as valid by QC.

### (C) Match & Merge
- Build one flat QC lookup table: every QC row expressed as product code, lot, bag range, matching mode, and its QC result columns.
- Add placeholder rows for lots QC recorded but Spectro never physically measured, since Spectro samples lots by alternating through a batch rather than testing every one — a confirmed lab practice, not an assumption. This currently applies to Mode B only.
- Match each Spectro row against the QC lookup using product code, lot, and bag-range containment (not exact bag match, since Spectro sometimes takes several separate readings within one QC-recorded bag range).
- Oversize (OS) status for matched rows comes from Spectro's own data; for placeholder rows (no Spectro reading), OS status is read directly from QC's remarks.

### (D) Output
- Rename columns to clear, final report names.
- Clean up text formatting (uppercase names/customer values, whole-number formula IDs).
- Set the final column order (see Section 5).
- Sort rows: reference rows first, then grouped by lot number, then by bag number.
- Apply color formatting (see Section 6).
- Save the final Excel file to a dedicated output folder, automatically named with a version label and the exact date and time created, using Philippine time.

---

## 5. Final Output Structure

**Column order:**
1. Color Simulation
2. Lot Number
3. Product Code
4. Bag Number
5. Oversize
6. Date Time
7. Spectro measurement columns (ΔE*00, L*, C*, h°, a*, b*, ΔL*, ΔC*, ΔH*, Δa*, Δb*, Color Offset)
8. Spectro Judgement
9. Final QC Status
10. Evaluated On
11. Evaluated By
12. Customer
13. Formula ID
14. Match Status

**Sheets (tabs):**
1. **RAW_DATA** — all data, unfiltered.
2. **COMPLETE_DATA** — all data except reference rows and QC-only (no Spectro reading) rows. This is the clean dataset intended for testing and training.
3. **REFERENCE_DATA** — reference rows only (STD/LIGHT/DARK). For lookup reference; not used in testing or training.
4. **INCOMPLETE_DATA** — QC-only rows (QC has a result but Spectro never physically measured that lot). These need lab clarification and are excluded from COMPLETE_DATA.
