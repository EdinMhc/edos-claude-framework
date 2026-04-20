# Feature: Billing Migration
**Status:** IN_PROGRESS
**Started:** 2026-04-19
**Completed:** —

---

## Requirements

Migrate all billing features from the Next.js TMS (`joyride-tms`) to the .NET API (`joyride-tms-api`). Backend only.

### Phase 1 — Infrastructure Services (S3, Email, PDF)
- Add NuGet packages: `AWSSDK.S3`, `QuestPDF`, `Resend` to `JoyRide.Infrastructure`
- Add `AwsSettings` + `ResendSettings` sections to `appsettings.json`
  - AWS: Region `us-west-1`, Bucket `tms-load-documents`
  - Resend: from `EnvConfig.txt`, test email `edinmuhic00@gmail.com`
- Create `IS3Service` / `S3Service` — upload/delete files on S3
- Create `IEmailService` / `ResendEmailService` — send invoice PDF via Resend
- Create `IInvoicePdfService` / `QuestPdfInvoiceService` — generate PDF from invoice data (QuestPDF community license)
- Register all three in `DependencyInjection.cs` (S3/Email as Scoped, PDF as Singleton)

### Phase 2 — Customer/Load Extensions + Enums + Permissions
- Add 4 new enums: `ContractStatus`, `LaneFrequencyUnit`, `InvoiceItemType`, `InvoiceStatus`
- Add 4 new permissions to `Permissions.cs`: `CustomerContractsRead/Write`, `InvoicesRead/Write` (update `All[]`)
- Extend `Customer` entity: `BillingEmail`, `BillingRequiresRC`, `BillingRequiresBOL`, `BillingRequiresPOD`
  - Customer already has: CreditLimit, PaymentTerms, McNumber, DotNumber, TaxId — no changes needed
- Extend `Load` entity: `BaseRate (decimal)`, `ContractId (Guid?)`, `LaneId (Guid?)`
- Update CustomerConfiguration + LoadConfiguration for new columns
- Update `CustomerDto` / `CreateCustomerDto` / `UpdateCustomerDto` + `CustomerService`
- Update `LoadDto` / `LoadService` for `BaseRate`
- Migration: `AddBillingFieldsToCustomerAndLoad`

### Phase 3 — Customer Contracts
- New entities: `CustomerContract`, `ContractLane`, `ContractRate`, `ContractDocument`
- EF configs (snake_case, NEWID(), GETUTCDATE(), enum as string)
  - Cascade caution: `ContractDocument → Contract` = `NoAction`; `ContractLane → CustomerLocations` = `ClientSetNull`
  - Unique index on `(tenant_id, contract_number)` in `customer_contracts`
- Add 4 DbSets + `SaveChangesAsync` update hooks for `CustomerContract` and `ContractRate`
- Add FK configs + nav props from `Load → CustomerContract/ContractLane`
- DTOs: `CustomerContractDto`, `ContractLaneDto`, `ContractRateDto`, `ContractDocumentDto`, `CreateCustomerContractDto`
- `ICustomerContractService` / `CustomerContractService` (inject `ApplicationDbContext`, `IS3Service`)
  - `CreateContractAsync`: check ContractNumber uniqueness per tenant, create contract + lanes + rates atomically
  - `UploadDocumentAsync`: S3 key = `contracts/{contractId}/{newGuid}/{fileName}`
  - `DeleteContractAsync`: delete S3 objects first, then contract (cascade removes lanes/rates in DB)
- `CustomerContractErrors` static class in `Error.cs`
- `CustomerContractsController`: route `api/customers/{customerId}/contracts`, + `POST /{id}/documents` (multipart)
- Migration: `AddCustomerContracts`

### Phase 4 — Invoices
- New entities: `Invoice`, `InvoiceItem`
- EF configs — cascade caution: `Invoice.LoadId` and `Invoice.CustomerId` = `NoAction`; `InvoiceItem.InvoiceId` = `NoAction` (service-layer cascade)
- Update `IInvoicePdfService` to accept `InvoiceDto` (drop temporary `InvoicePdfModel`)
- DTOs: `InvoiceDto`, `InvoiceItemDto`, `CreateInvoiceDto`, `SetInvoiceItemsDto`, `CreateInvoiceItemDto`, `UpdateInvoiceStatusDto`
- `IInvoiceService` / `InvoiceService` (inject `ApplicationDbContext`, `IS3Service`, `IEmailService`, `IInvoicePdfService`)
  - `GetReadyForInvoicingAsync`: load Status=Completed + no invoice + customer document requirements (RC/BOL/POD = Approved)
  - `CreateInvoiceAsync`: verify load + customer, check no duplicate, compute DueDate from PaymentTerms string, InvoiceNumber = load.LoadNumber
  - `SetItemsAsync`: replace all items, Amount = Qty × UnitPrice, recalculate TotalAmount
  - `UpdateStatusAsync`: parse InvoiceStatus enum, require PartiallyPaidAmount if PartiallyPaid
  - `SendInvoiceAsync`: generate PDF → S3 upload → Resend email → status = Sent
- `InvoiceErrors` static class in `Error.cs`
- `InvoicesController`: `GET /ready` MUST be declared before `GET /{id}`
- Migration: `AddInvoices`

---

## Files Modified

| File | What changed |
|---|---|
| `src/JoyRide.Infrastructure/JoyRide.Infrastructure.csproj` | Added AWSSDK.S3 3.7.400.1, QuestPDF 2024.10.4, Resend 0.4.0 |
| `src/JoyRide.API/appsettings.json` | Added `Aws` and `Resend` config sections |
| `src/JoyRide.Application/Interfaces/Services/IS3Service.cs` | New — upload/delete/deleteMany interface |
| `src/JoyRide.Application/Interfaces/Services/IEmailService.cs` | New — SendInvoiceAsync interface |
| `src/JoyRide.Application/Interfaces/Services/IInvoicePdfService.cs` | New — Generate(InvoicePdfModel) interface |
| `src/JoyRide.Application/DTOs/Billing/InvoicePdfModel.cs` | New — temporary DTO (replaced by InvoiceDto in Phase 4) |
| `src/JoyRide.Infrastructure/Settings/AwsSettings.cs` | New — binds `Aws` config section |
| `src/JoyRide.Infrastructure/Settings/ResendSettings.cs` | New — binds `Resend` config section |
| `src/JoyRide.Infrastructure/Services/S3Service.cs` | New — S3 upload/delete, AmazonS3Client created per-call |
| `src/JoyRide.Infrastructure/Services/ResendEmailService.cs` | New — sends invoice PDF via Resend API |
| `src/JoyRide.Infrastructure/Services/QuestPdfInvoiceService.cs` | New — QuestPDF community license invoice generator (singleton) |
| `src/JoyRide.Infrastructure/DependencyInjection.cs` | Registered AwsSettings, ResendSettings, IResend, IS3Service, IEmailService, IInvoicePdfService |
| `src/JoyRide.Domain/Enums/ContractStatus.cs` | New — Draft, Active, Expired, Terminated |
| `src/JoyRide.Domain/Enums/LaneFrequencyUnit.cs` | New — PerWeek, PerMonth, PerYear |
| `src/JoyRide.Domain/Enums/InvoiceItemType.cs` | New — Freight, FuelSurcharge, Detention, Lumper, Accessorial, Other |
| `src/JoyRide.Domain/Enums/InvoiceStatus.cs` | New — Draft, Sent, Paid, PartiallyPaid, Void, Overdue |
| `src/JoyRide.Domain/Constants/Permissions.cs` | Added CustomerContractsRead/Write + InvoicesRead/Write to constants and All[] |
| `src/JoyRide.Infrastructure/Identity/Customer.cs` | Added BillingEmail, BillingRequiresRC, BillingRequiresBOL, BillingRequiresPOD |
| `src/JoyRide.Infrastructure/Identity/Load.cs` | Added BaseRate, ContractId, LaneId |
| `src/JoyRide.Infrastructure/Persistence/Configurations/CustomerConfiguration.cs` | Added EF mappings for 4 billing columns |
| `src/JoyRide.Infrastructure/Persistence/Configurations/LoadConfiguration.cs` | Added EF mappings for base_rate, contract_id, lane_id |
| `src/JoyRide.Application/DTOs/Accounting/CustomerDto.cs` | Added billing fields to CustomerDto, CreateCustomerDto, UpdateCustomerDto |
| `src/JoyRide.Application/DTOs/Operations/LoadDto.cs` | Added BaseRate, ContractId, LaneId to LoadDto, CreateLoadDto, UpdateLoadDto |
| `src/JoyRide.Infrastructure/Services/CustomerService.cs` | Updated Create, Update, ToDto for billing fields |
| `src/JoyRide.Infrastructure/Services/LoadService.cs` | Updated Create, Update, ToDto for BaseRate/ContractId/LaneId |
| `src/JoyRide.Infrastructure/Persistence/Migrations/*_AddBillingFieldsToCustomerAndLoad.cs` | Migration: 7 new columns across customers + loads tables |

---

## Decisions Made

- **PDF library**: QuestPDF (community license) — generates server-side PDFs without browser dependency
- **Email provider**: Resend.com — already has API key from existing Next.js config
- **S3 bucket**: Reuse existing `tms-load-documents` bucket (region `us-west-1`); not creating a new one
- **S3 client lifecycle**: `AmazonS3Client` created per-call inside `S3Service` (not singleton) to avoid connection pool issues with credentials rotation
- **PDF service lifecycle**: Singleton — `QuestPdfInvoiceService.Generate()` is stateless and thread-safe
- **InvoicePdfModel**: Temporary DTO created in Phase 1 so `IInvoicePdfService` compiles without `InvoiceDto`. Replaced by `InvoiceDto` in Phase 4; `InvoicePdfModel.cs` is deleted at that point.
- **Cascade strategy**: SQL Server does not allow multiple cascade paths to the same table. `invoice_items` FKs to both `tenants` (Cascade) and `invoices` (NoAction). Items are deleted in the service layer before invoice deletion. Same pattern for `ContractDocument`.
- **ContractNumber uniqueness**: Unique index on `(tenant_id, contract_number)` — enforced at DB level, also validated in service for a clean error message.
- **InvoiceNumber = LoadNumber**: Matches Next.js behavior. Unique per tenant enforced by DB index `(tenant_id, invoice_number)`.
- **`GET /ready` route ordering**: Declared before `GET /{id:guid}` in controller to avoid ASP.NET Core routing ambiguity between literal and parameterized segments.
- **Due date calculation**: Derived from `customer.PaymentTerms` string (`due_on_receipt`/`within_30_days`/`within_60_days`/`within_90_days`), defaulting to +30 days.
- **`AttachedDocumentIds`**: Stored as a JSON string column in `invoices` (not a separate join table) for simplicity — these are references to `LoadDocuments`, not owned records.
- **Contract document upload**: Multipart form upload directly to the API; service uploads to S3 and stores the URL + key. No pre-signed URL flow.
- **`BillingEmail ?? Email` fallback**: `SendInvoiceAsync` tries `customer.BillingEmail` first, falls back to `customer.Email`. Returns `InvoiceErrors.NoBillingEmail` if both null.

---

## Session Notes
- 2026-04-19: Feature created. Explored existing Next.js billing system and .NET codebase patterns. Full 4-phase implementation plan designed and saved to `C:\Users\Ednmh\.claude\plans\silly-orbiting-engelbart.md`. All architectural decisions confirmed. Ready to begin implementation at Phase 1.
- 2026-04-20: Phase 1 complete. Added NuGet packages (AWSSDK.S3, QuestPDF, Resend 0.4.0). Created IS3Service, IEmailService, IInvoicePdfService interfaces. Created InvoicePdfModel temp DTO. Implemented S3Service, ResendEmailService, QuestPdfInvoiceService. Registered all in DI. Build: 0 errors, 0 warnings. Ready for Phase 2.
- 2026-04-20: Phase 2 complete. Added 4 enums (ContractStatus, LaneFrequencyUnit, InvoiceItemType, InvoiceStatus). Added 4 permissions (CustomerContractsRead/Write, InvoicesRead/Write). Extended Customer entity (4 billing fields) and Load entity (BaseRate, ContractId, LaneId). Updated EF configs, all DTOs, CustomerService, LoadService. Migration AddBillingFieldsToCustomerAndLoad created. Build: 0 errors, 0 warnings. Ready for Phase 3.

---

## Test Checklist

End-to-end verification sequence:
1. `dotnet build JoyRide.TMS.sln` — 0 errors
2. All 3 migrations apply cleanly with `dotnet ef database update`
3. `PATCH /api/customers/{id}` — verify `billingRequiresPOD`, `billingEmail` fields in response
4. `POST /api/customers/{customerId}/contracts` — create contract with lanes + rates
5. `POST /api/customers/{customerId}/contracts/{id}/documents` — upload file, verify S3 URL returned
6. Complete a load for the customer with an approved POD document
7. `GET /api/invoices/ready` — load appears
8. `POST /api/invoices` — invoice created as Draft with correct DueDate
9. `POST /api/invoices/{id}/items` — add items, verify `totalAmount` recalculated
10. `POST /api/invoices/{id}/send` — email arrives at `edinmuhic00@gmail.com`, `pdfUrl` populated, status = Sent

---

## Summary

*(written at WRAP UP)*
