# Work Summary - 15-05-2026

## Session Info
- **Date:** Friday, 15 May 2026
- **Carried Over From:** 14-05-2026
- **Agents / areas active today:** Invoicing, Customer portal

## Today's Work

### 10:12 - Invoice PDF shows the wrong currency symbol for USD customers

<!-- Part 1: Technical Details -->
**Request:** Invoices for customers billed in USD print the AED symbol in the totals block. Fix the PDF so it uses the invoice currency.

**Files Modified:**
- `templates/invoice_pdf.html`: totals block reads `invoice.currency` instead of the company default.
- `tests/test_invoice_pdf.py`: two new cases (USD invoice, AED invoice).

**Root cause:** the totals block formatted amounts with the company default currency; line items already used the invoice currency, so only the totals were wrong.

**Verification:** `pytest tests/test_invoice_pdf.py` 6 passed; generated PDFs for INV-0412 (USD) and INV-0413 (AED) and checked both by eye.

**Status:** Completed
**Commit:** 3f9c2ab

<!-- Part 2: Human Summary (MANDATORY) -->
> **Human Summary:**
> Task: Fixed invoice printouts so totals show the customer's own billing currency
> Task summary: Fixed a problem where invoices for customers billed in US dollars showed the dirham symbol next to the subtotal, tax and grand total, even though each line item showed dollars correctly. The printout now uses the currency of the invoice itself for every amount, so a dollar invoice shows dollars throughout and a dirham invoice is unchanged. Two automatic checks were added so the mistake cannot come back unnoticed, and printouts for one dollar invoice and one dirham invoice were reviewed by hand to confirm the result. Customers billed in foreign currencies will no longer receive invoices that look inconsistent, which removes a recurring reason for finance queries and resent documents.
> Date: 15 May 2026
> Duration:

---

### 11:40 - [Invoicing] Testing: currency symbol on invoice PDFs

<!-- Part 1: Technical Details -->
**Request:** Test the 10:12 fix in the running app.

**Test cases:**
| # | Case | Result |
|---|------|--------|
| 1 | USD invoice, totals show $ | Pass |
| 2 | AED invoice, totals show AED | Pass |
| 3 | EUR invoice with discount row | Fail (B1), then Pass |
| 4 | Draft invoice preview | Pass |
| 5 | Browser console and network during download | Pass |

**Bug log:**
| ID | Steps | Expected | Actual | Severity | Status | Fix |
|----|-------|----------|--------|----------|--------|-----|
| B1 | EUR invoice with a discount row, download PDF | EUR on discount row | AED on discount row | Medium | Verified | discount row now uses `invoice.currency` (templates/invoice_pdf.html) |

**Cycles:** 2. **Final result:** full pass, no console errors, no failed requests.

**Status:** Completed

<!-- Part 2: Human Summary (MANDATORY) -->
> **Human Summary:**
> Task: Tested the invoice currency fix across currencies, discounts and draft previews
> Task summary: Tested the invoice printout fix in the running application across five situations: dollar and dirham invoices, a euro invoice with a discount line, the preview of an unsaved draft, and the download itself. One further problem was found, where the discount line on a euro invoice still showed the dirham symbol. It was fixed straight away and the full set of checks was run again, this time passing completely with no errors on screen or in the background. The invoice printout now shows the correct currency on every line, including discounts, for all the currencies the business bills in, so finance can send foreign-currency invoices without checking each one by hand.
> Date: 15 May 2026
> Duration:

---

### 15:05 - Customer portal: show the last five payments on the dashboard

<!-- Part 1: Technical Details -->
**Request:** Add a "Recent payments" card to the customer portal dashboard listing the last five payments.

**Status:** In Progress

<!-- Part 2: Human Summary (MANDATORY) -->
> **Human Summary:**
> Task:
> Task summary:
> Date: 15 May 2026
> Duration:

---

## End of Day Summary
- **Completed:** Invoice PDF currency fix; testing of the fix
- **In Progress:** Customer portal recent payments card
- **Blocked:** -
- **Tomorrow:** Finish the recent payments card and test it
