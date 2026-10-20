# Invoice PDF Generation & Delivery Automation

> **Architecture:** See [ARCHITECTURE.md](ARCHITECTURE.md) for the infographic, detailed flow, and trust boundaries.

An n8n workflow template that validates an invoice request, calculates an INR invoice total using a configurable-looking but currently hard-coded 18% GST rate, renders a PDF, stores the invoice record and file, and emails the PDF to the submitted client address. It tracks duplicate requests and has explicit persistence, PDF, and email failure branches.

> **Important:** This workflow can create and email financial documents. The source export does **not** show authentication configured on the webhook, despite its original node label. Do not expose or activate it until authentication, authorization, tax rules, numbering, recipient verification, and accounting controls are reviewed.

## Project snapshot

- **Category:** n8n automation, invoice document generation, PDF delivery, business operations
- **Workflow size:** 35 nodes, 28 connection groups in the supplied export
- **Integrations:** Google Sheets, Google Drive, Gmail, Slack, PDFBolt, n8n webhook/respond nodes
- **Invoice currency:** INR (₹) in the current HTML template
- **Tax assumption:** GST is fixed at 18% in the source; it is not a tax engine and does not validate applicability
- **Payment:** The PDF prints payment-method instructions, but this workflow does not process payments
- **Public export:** `Workflow.json` has credentials and source account identifiers scrubbed

## Workflow canvas preview

![n8n editor view: invoice generation workflow](assets/workflow-canvas.png)

> This editor screenshot is illustrative only; it does not prove secure configuration, live credentials, or production readiness.

## Workflow behavior

1. Accepts a POST invoice request with `request_id`, client name/email, service description, and amount.
2. Normalizes and validates the fields, including an amount greater than zero, up to 10,000,000, with at most two decimal places. Invalid requests return HTTP 400.
3. Looks up the request ID in Google Sheets. Existing requests return a duplicate response without starting a second invoice email; lookup failures are routed separately.
4. For a new request, returns an HTTP **202 accepted** acknowledgement, then continues processing. The sanitized export corrects an inaccurate early response that previously claimed the invoice had already been sent.
5. Generates an invoice number, issue date and 15-day due date, calculates GST/total, and reserves a `Processing` row in Sheets.
6. Escapes client-facing text for the HTML invoice, renders a PDF, and validates that the generated binary is a PDF.
7. Uploads the file to the configured Drive folder, updates the record to pending email, and sends the PDF to the submitted client email.
8. Marks the invoice sent or flags the email for manual checking, then notifies Slack on relevant completion/failure paths.

## Request fields

| Field | Required | Validation in source template |
|---|---:|---|
| `request_id` | Yes | 8–100 characters: letters, digits, underscore, hyphen |
| `client_name` | Yes | 2–120 characters |
| `client_email` | Yes | Basic email format, up to 254 characters |
| `service` | Yes | 2–500 characters |
| `amount` | Yes | Positive numeric/string value, max 10,000,000, up to two decimal places |

Use fake names, addresses, and values in portfolio demos. Never use real invoice/customer information in public screenshots or test runs.

## Important business and financial assumptions

- The template displays **INR (₹)** and applies **18% GST** unconditionally. Validate GST registration, supply type, place of supply, exemptions, invoice fields, and applicable rate with a qualified accounting/tax professional before issuing real invoices.
- The invoice number currently combines the date with the n8n execution ID. That may expose internal operational identifiers and is not guaranteed to meet statutory sequencing/uniqueness rules. Replace it with a controlled, auditable invoice-number allocator.
- The due date defaults to 15 days from the issue date. Confirm this matches the actual contract and applicable terms.
- The invoice HTML includes generic sample business branding and `billing@example.com`; replace all sample branding/contact information with authorized details before generating a real PDF.
- The service description is carried into an invoice line item; validate it against the approved service/order and amount. This template does not reconcile against a contract or payment ledger.

## Import and configure

1. Import `Workflow.json` into a test n8n workspace.
2. **Before activation, add strong webhook authentication.** The source export contains no webhook auth setting. Add rate limiting, request-size caps, abuse monitoring, and caller authorization; do not rely on the endpoint path as a secret.
3. Bind least-privilege Google Sheets, Google Drive, Gmail, Slack, and PDFBolt credentials in n8n. The public file intentionally has no credential bindings.
4. Replace the Google Sheet/tab, Drive folder, and Slack channel placeholders with private resources you control. Ensure the Sheet is not publicly shared.
5. Replace sample sender branding/email, configure the correct invoice sequence, tax/currency treatment, contractual payment terms, and receipt/status process.
6. Verify client identity and email address against an authorized source before allowing an invoice to be emailed. A caller-supplied email is not proof the recipient is entitled to receive a financial document.
7. Test duplicates, invalid amounts, sheet lookup/reservation failures, PDF generation failure, Drive upload failure, Gmail failure, and retry behavior using synthetic records. Check for duplicate invoices and duplicate emails under concurrent requests.
8. Activate only after accounting, privacy, authorization, and communication controls have been reviewed for your jurisdiction and business.

## Security and privacy

This sanitized export removes credential objects, source webhook IDs, original Google Sheet and Drive identifiers, Slack channel identifiers, the n8n instance ID, source brand references, and non-example email addresses. These removals do **not** revoke any credential or undo a prior disclosure.

- **Webhook security:** Authentication is missing in the source export. This is a release blocker because an unauthenticated caller could trigger file creation and client-facing email. Add and test actual authentication and authorization.
- **Recipient control:** Verify the request source, client, amount, and recipient. Prevent arbitrary callers from emailing invoices to third parties.
- **Financial data:** Client names, emails, service descriptions, amounts, invoice numbers, and Drive files are sensitive. Restrict Gmail/Drive/Sheets access, execution history, retention, and backups.
- **Duplicate/idempotency safety:** Request-ID lookup before reserve reduces accidental duplicates but may not be atomic. Concurrent identical requests can race; use an atomic unique-key datastore/lock for strict idempotency.
- **HTML safety:** The workflow escapes text interpolated into invoice HTML. Keep this escaping if editing templates and never render untrusted markup.
- **File handling:** Validate generated PDF MIME type and apply appropriate file-size, storage permissions, retention, and download-sharing controls.
- **Secret management:** Use n8n's credential vault and least-privilege service accounts. Rotate credentials if exposed in any public repo/history.
- **No payment execution:** No payment is collected or transferred by this workflow. Payment text in the invoice is informational and must be configured and verified separately.

## Known limitations

- Fixed 18% GST, INR currency, 15-day terms, and sample invoice branding are defaults, not validated tax/accounting advice.
- Invoice number generation uses a workflow execution ID and is not a compliant numbering service by itself.
- Google Sheets lookup-then-reserve may not prevent race conditions under parallel traffic.
- The workflow emails the invoice automatically after the requested steps; sender must authorize that behavior and verify the recipient before production.
- The 202 acknowledgement is sent before processing finishes. There is no separate status-query endpoint; repeat the same request ID only after checking the stored record.

## Suggested freelancing portfolio description

> Built an n8n invoice-document workflow with request validation, duplicate-request detection, INR/GST calculations, HTML escaping, PDF generation, Drive storage, client email delivery, and explicit failure/manual-check states. Includes idempotent retry considerations and a credential-scrubbed public template. Production use requires webhook authentication, recipient authorization, compliant invoice numbering, and jurisdiction-specific tax/accounting validation.

## License

Publish only if you have rights to redistribute the workflow, invoice design, and all embedded configuration. Review n8n, third-party node, and provider terms before publication.


## Synthetic request and response examples

- [Sample request](sample/request.json)
- [Sample successful response](sample/response.json)

These examples use synthetic demonstration data and show the successful route only. HTTP status codes are returned separately by the workflow. Configure authentication and credentials in your own n8n instance before activation.
