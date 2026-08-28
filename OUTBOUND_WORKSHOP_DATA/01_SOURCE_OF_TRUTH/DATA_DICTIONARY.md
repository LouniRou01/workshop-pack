# Source-of-truth data

The source-of-truth files contain synthetic account and contact profiles used throughout the workshop. Every contact is linked to an account through `account_id`; `company_name` and related company fields are also included on contact rows for participant usability.

## Accounts

- `accounts.csv` contains 90 synthetic accounts.
- `account_id` is the stable account identifier.
- Domains use the reserved `.example` suffix.

## Contacts

- `contacts.csv` contains 1,000 synthetic contacts.
- Every contact has a valid `account_id` and matching company fields.
- `contact_id` is the stable contact identifier.
- Contact and CRM-style fields include role, seniority, persona fit, email status, lead status, sequence status, source, and next step.

## Relationships

- `contacts.account_id` joins to `accounts.account_id`.
- `02_SOURCE_SYSTEMS/crm_state.csv` contains operational account and contact state.
- `02_SOURCE_SYSTEMS/signals.csv` contains research signals linked to the original scenario accounts.

All records are fictional and safe for workshop use.
