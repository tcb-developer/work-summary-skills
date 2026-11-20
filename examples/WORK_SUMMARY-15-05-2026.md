# Work Summary - 15-05-2026

## Session Info
- **Date:** Friday, 15 May 2026
- **Carried Over From:** 14-05-2026

## Today's Work

### 10:12 - [Invoicing] Invoice PDF shows the wrong currency symbol for USD customers

**Status:** Completed
**Request:** Invoices for customers billed in USD print the AED symbol in the totals block. Make the totals use the invoice currency.
**Touched:** `templates/invoice_pdf.html:88-112` (totals block), `tests/test_invoice_pdf.py` (`test_totals_currency_usd`, `test_totals_currency_aed`)
**Cause:** the totals block formatted amounts with the company default currency; line items already used `invoice.currency`.
**Change:** totals block reads `invoice.currency`; two tests added.
**Decisions:** left the company default as the fallback for invoices with no currency set (legacy imports).
**Verify:** `pytest tests/test_invoice_pdf.py` -> 6 passed; PDFs for INV-0412 (USD) and INV-0413 (AED) checked by eye.
**Commit:** 3f9c2ab on main
**Next:** -

> **Human Summary:**
> Task: Fixed invoice printouts so totals show the customer's own billing currency
> Task summary: Fixed a problem where invoices for customers billed in US dollars showed the dirham symbol next to the subtotal, tax and grand total, even though each line item showed dollars correctly. The printout now uses the currency of the invoice itself for every amount, so a dollar invoice shows dollars throughout and a dirham invoice is unchanged. Two automatic checks were added so the mistake cannot come back unnoticed, and printouts for one dollar invoice and one dirham invoice were reviewed by hand to confirm the result. Customers billed in foreign currencies will no longer receive invoices that look inconsistent, which removes a recurring reason for finance queries and resent documents.
> Date: 15 May 2026
> Duration:

---

### 11:40 - [Invoicing] Testing: currency symbol on invoice PDFs

**Status:** Completed
**Request:** Test the 10:12 fix in the running app.
**Cases:** USD totals Pass; AED totals Pass; EUR with discount row Fail (B1) then Pass; draft preview Pass; console and network during download Pass.
**Bug log:** B1 | EUR invoice with a discount row, download PDF | expected EUR on discount row | got AED | Medium | Verified | discount row now uses `invoice.currency` in `templates/invoice_pdf.html:131` (`discount_row`)
**Verify:** 2 cycles; final full pass, no console errors, no failed requests.
**Commit:** 7d01e5a on main

> **Human Summary:**
> Task: Tested the invoice currency fix across currencies, discounts and draft previews
> Task summary: Tested the invoice printout fix in the running application across five situations: dollar and dirham invoices, a euro invoice with a discount line, the preview of an unsaved draft, and the download itself. One further problem was found, where the discount line on a euro invoice still showed the dirham symbol. It was fixed straight away and the full set of checks was run again, this time passing completely with no errors on screen or in the background. The invoice printout now shows the correct currency on every line, including discounts, for all the currencies the business bills in, so finance can send foreign-currency invoices without checking each one by hand.
> Date: 15 May 2026
> Duration:

---

### 15:05 - [Portal] Show the last five payments on the customer dashboard

**Status:** In Progress
**Request:** Add a "Recent payments" card to the customer portal dashboard listing the last five payments.
**Touched:** `portal/dashboard.py:40-75` (`get_dashboard_context`), `portal/templates/dashboard.html`
**Decisions:** reuse the existing payments query with a limit; no new endpoint.
**Next:** card markup done; still need the empty state and a test.

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
- **Tomorrow:** Finish the recent payments card (empty state, test)
