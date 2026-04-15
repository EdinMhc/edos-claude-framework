# Feature: Billing Migration
**Status:** PLANNING
**Started:** 2026-04-14
**Completed:** —

---

## Requirements

Migrate all billing features from the Next.js TMS (`joyride-tms`) to the .NET API (`joyride-tms-api`). Backend only — no frontend work.

### Entities to implement

**1. Customer (extend existing)**
- Existing `Customers` table already exists in .NET. Add missing billing fields:
  - `creditLimit` (decimal, USD)
  - `paymentTerms` (text)
  - `billingEmail` (text)
  - `billingRequiresRC` (bool) — Rate Confirmation required before invoicing
  - `billingRequiresBOL` (bool) — Bill of Lading required before invoicing
  - `billingRequiresPOD` (bool) — Proof of Delivery required before invoicing
  - `mcNumber`, `dotNumber`, `taxId` (text)
  - `preferredContactMethod` (enum: email, phone, none)
  - `vic`, `doNotEmail`, `doNotCall` (bool flags)
- Customer types already may exist — confirm enum matches: `shipper, broker, consignee, carrier, other`

**2. CustomerContracts** (new table)
- `id`, `tenantId`, `customerId`
- `name`, `contractNumber`
- `status` (enum: draft, active, expired, terminated)
- `startDate`, `endDate`
- `paymentTerms`, `notes`
- Relations: rates (many), documents (many), lanes (many)

**3. ContractLanes** (new table)
- `id`, `contractId`
- `laneCode` (e.g. "NYC-CHI")
- `laneFrequency` (int), `laneFrequencyUnit` (enum: day, week, month, year)
- `pickupLocationId`, `deliveryLocationId` (references to CustomerLocations)

**4. ContractRates** (new table)
- `id`, `contractId`, `laneId` (nullable)
- `type` (enum: linehaul, detention, layover, fuel_surcharge, mileage, per_stop, weight, lumper, other)
- `description`, `amount` (decimal)

**5. ContractDocuments** (new table)
- `id`, `contractId`
- `documentUri`, `fileName`, `fileSize`, `mimeType`, `s3Key`
- Standard audit fields

**6. Invoices** (new table)
- `id`, `tenantId`, `loadId`, `customerId`
- `invoiceNumber` (unique per tenant, = load number)
- `pdfUrl`, `pdfKey` (S3)
- `attachedDocuments` (JSON array of document references)
- `status` (enum: draft, sent, paid, partially_paid, disputed, voided)
- `dueDate`, `totalAmount` (decimal)
- `partiallyPaidAmount` (decimal)
- `notes`
- Standard audit fields

**7. InvoiceItems** (new table)
- `id`, `tenantId`, `loadId`, `invoiceId`, `rateId` (nullable)
- `type` (same enum as ContractRates type)
- `description`, `quantity` (decimal, default 1), `unitPrice` (decimal), `amount` (decimal)
- Standard audit fields

**8. Load (extend existing)**
- Add `baseRate` (decimal), `contractId` (nullable FK), `laneId` (nullable FK)

### Endpoints to implement

**Customers**
- `PATCH /api/customers/{id}` — extend update to include new billing fields
- Existing customer endpoints already handle create/read/delete

**Customer Contracts**
- `GET /api/customers/{customerId}/contracts` — list contracts (filter by status)
- `GET /api/customers/{customerId}/contracts/{contractId}` — single contract
- `POST /api/customers/{customerId}/contracts` — create contract with lanes, rates, documents
- `DELETE /api/customers/{customerId}/contracts/{contractId}` — delete (cascades)

**Invoices**
- `GET /api/invoices` — list with filters (status[], customerIds[], pagination)
- `GET /api/invoices/{id}` — single invoice with items + load data
- `GET /api/invoices/ready` — loads eligible for invoicing (completed, no invoice, documents satisfied)
- `POST /api/invoices` — create invoice
- `PATCH /api/invoices/{id}` — update status / fields
- `POST /api/invoices/{invoiceId}/items` — set invoice items (upsert)

### Business Logic

- **Invoice total** = sum of (quantity × unitPrice) for all items
- **Ready for invoicing**: Load must be `completed`, no existing invoice, all customer-required document types must have an approved document
- **Due date calculation**: based on paymentTerms string → `due_on_receipt` = 0 days, `within_30_days` = +30d, `within_60_days` = +60d, `within_90_days` = +90d
- **Partial payments**: `partially_paid` status requires `partiallyPaidAmount` to be set
- **Rate-locked items**: invoice items linked to a `rateId` cannot have quantity/unitPrice edited

### Permissions required
`INVOICES_READ`, `INVOICES_WRITE`, `CUSTOMERS_READ`, `CUSTOMERS_WRITE`, `CUSTOMER_CONTRACTS_READ`, `CUSTOMER_CONTRACTS_WRITE`

### Open questions (to clarify before coding)
- [ ] PDF generation: should the API generate invoice PDFs, or just return data for the frontend?
- [ ] S3: is AWS S3 already configured in the .NET project?
- [ ] Email: is there an email service already set up in the .NET project?
- [ ] Customer type enum: does the existing .NET `Customers` table already have a `type` column matching `shipper/broker/consignee/carrier/other`?
- [ ] Load billing fields (`baseRate`, `contractId`, `laneId`): add to existing Load entity via migration?
- [ ] Contract document uploads: multipart form or separate upload endpoint with S3 URL reference?

---

## Files Modified

| File | What changed |
|---|---|

---

## Decisions Made

---

## Session Notes
- 2026-04-14: Feature created. Explored existing Next.js billing system. Requirements drafted. Awaiting clarification on 6 open questions before coding begins.

---

## Test Checklist

---

## Summary

