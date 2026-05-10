# AAX Data Entities

Custom data entities for Microsoft Dynamics 365 Finance and Operations that simplify data import/export operations by removing the requirement for party number knowledge and by providing unfiltered access to packing slip journals.

**Publisher:** AtomicAX.com  
**Model:** AAX Data Entities  
**Layer:** 14 (ISV)

## Overview

The standard D365 F&O data entities for customer/vendor contacts require knowledge of the global address book party number, which is often unavailable during data migrations from legacy systems. This package provides wrapper entities that accept the customer or vendor account number directly and resolve the party number internally.

Additionally, the standard packing slip entities apply implicit filters that may exclude records. This package exposes packing slip data with no such restrictions.

## Data Entities

### Contact Entities

| Entity | Public Name | Module | Description |
|--------|-------------|--------|-------------|
| AAXCustomerContactInfoEntity | CustomerContactInfo | Accounts Receivable | Import/export customer electronic contact information (phone, email, URL, etc.) using only the customer account number. Automatically resolves party number and location ID. |
| AAXCustomerContactPersonEntity | — | Accounts Receivable | Import/export customer contact persons using only the customer account number. Wraps smmContactPersonV2Entity with automatic party resolution. |
| AAXVendorContactInfoEntity | VendorContactInfo | Accounts Payable | Import/export vendor electronic contact information using only the vendor account number. Includes fallback lookup by KnownAs field if the vendor account is not found directly. |
| AAXVendorContactPersonEntity | — | Accounts Payable | Import/export vendor contact persons using only the vendor account number. Wraps smmContactPersonV2Entity with automatic party resolution. |

### Packing Slip Entities

| Entity | Public Name | Module | Description |
|--------|-------------|--------|-------------|
| AAXCustomerPackingSlipJournalHeaderEntity | CustomerPackingSlipJournalHeader | Accounts Receivable | Read-only entity exposing CustPackingSlipJour header records with delivery address, financial dimensions, and order taker. No implicit filters. |
| AAXCustomerPackingSlipJournalLinesEntity | CustomerPackingSlipJournalLines | Accounts Receivable | Read-only entity exposing CustPackingSlipTrans line records with item, quantity, and inventory dimensions. No implicit filters. |
| AAXCustomerPackingSlipJournalLineHistoryEntity | CustomerPackingSlipJournalLineHistory | — | Entity exposing packing slip line history (CustPackingSlipTransHistory) including Brazilian fiscal document data. |
| AAXCustomerPackingSlipsEntity (Composite) | — | — | Composite entity combining packing slip headers and lines for bulk export via the Data Management Framework. |

## Security Privileges

Each entity has corresponding security privileges for controlling access:

| Privilege | Access Level |
|-----------|-------------|
| AAXCustomerContactInfoEntityExport | Read |
| AAXCustomerContactInfoEntityImport | Write |
| AAXCustomerContactPersonsEntityExport | Read |
| AAXCustomerContactPersonsEntityImport | Write |
| AAXVendorContactInfoEntityExport | Read |
| AAXVendorContactInfoEntityImport | Write |
| AAXVendorContactPersonsEntityExport | Read |
| AAXVendorContactPersonsEntityImport | Write |
| AAXCustomerPackingSlipJournalHeaderEntityMaintain | Write |
| AAXCustomerPackingSlipJournalHeaderEntityView | Read |
| AAXCustomerPackingSlipJournalLinesEntityMaintain | Write |
| AAXCustomerPackingSlipJournalLinesEntityView | Read |
| AAXCustomerPackingSlipJournalLineHistoryEntityMaintain | Write |
| AAXCustomerPackingSlipJournalLineHistoryEntityView | Read |

## Module Dependencies

- ApplicationCommon
- ApplicationFoundation
- ApplicationPlatform
- ApplicationSuite
- ContactPerson
- Currency
- Dimensions
- Directory
- FiscalBooks
- Ledger
- Personnel / PersonnelCore / PersonnelManagement
- Retail
- SourceDocumentation / SourceDocumentationTypes
- Tax

## Key Design Decisions

- **Party number resolution:** Contact entities automatically resolve the party number from the customer/vendor account number during `initializeEntityDataSource`, so consumers never need to supply it.
- **Location ID auto-creation:** If no location ID is provided during import, the entities will find or create a DirPartyLocation for the party.
- **Vendor fallback lookup:** The vendor contact info entity includes a fallback that searches DirPartyTable.KnownAs when the vendor account doesn't resolve directly — useful for migrating data with alternate vendor identifiers.
- **Read-only packing slips:** The header and line packing slip entities are marked `IsReadOnly=Yes` to prevent accidental modification of posted transaction data.
- **Composite entity:** AAXCustomerPackingSlipsEntity provides a parent-child relationship (header-to-lines) for use with DMF export projects that need both in a single operation.

## Usage

### Data Management Framework (DMF)

1. Navigate to **Data management** workspace in D365 F&O.
2. Create a new **Export** or **Import** project.
3. Select the desired entity (e.g., "CustomerContactInfo" for customer contact information).
4. Map fields as needed — customer/vendor account number is the key identifier instead of party number.

### OData

The public entities are accessible via OData at:
- `CustomerContactInfos`
- `CustomerPackingSlipJournalHeaders`
- `CustomerPackingSlipJournalLines`
- `CustomerPackingSlipJournalLineHistories`
