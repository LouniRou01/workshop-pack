# Synthetic outbound data

All records in this folder are synthetic and use reserved `.example` domains.

## Entity relationships

- `accounts.csv` is the canonical account table.
- `contacts.csv` links contacts to accounts through `account_id`.
- `outbound_signals.csv` links research signals to accounts and, when available, contacts.
- `crm_accounts.csv` and `crm_contacts.csv` simulate operational CRM state.

Stable IDs are intentionally shared across files so agents can join records and explain their reasoning.
