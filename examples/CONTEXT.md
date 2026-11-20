# Project Context (for agents)
Updated: 2026-05-15
<!-- Durable, verified facts only. Under about 120 lines. Edit lines, do not append duplicates. -->

## Run / test
- App: `make dev` (port 8000). Tests: `pytest -q`; one file: `pytest tests/test_invoice_pdf.py`.
- PDFs render with the app's own template engine; preview at `/invoice/<name>/pdf` while `make dev` runs.

## Map (area -> path, entry points)
- Invoicing -> `billing/invoice.py` (`Invoice.submit`), PDF template `templates/invoice_pdf.html`.
- Portal -> `portal/dashboard.py` (`get_dashboard_context`), templates in `portal/templates/`.
- Auth -> `auth/login.py` (`login`), lockout counter on `auth/models.py` (`User.failed_logins`).

## Conventions and gotchas
- Money is formatted with `fmt_money(amount, currency)`; never pass the company currency for document totals.
- Legacy imported invoices can have an empty currency; code must fall back to the company default.
- Tests share one database; use the `clean_db` fixture or tests leak rows into each other.
