# Source-of-truth data

These files contain synthetic account and contact profiles. They are intended to describe what the entities are, not their changing CRM state.

## Accounts

- `accounts.csv` contains 90 synthetic accounts.
- `account_id` is the stable account identifier.
- `icp_fit` and `account_tier` are workshop classification fields defined by the targeting and outbound rules.
- CRM ownership, lifecycle, customer status, suppression, and pipeline state belong in `02_SOURCE_SYSTEMS/crm_state.csv` or related event files.
- Domains use the reserved `.example` suffix.

## Contacts

- `contacts.csv` contains 1,000 synthetic contacts.
- Every contact has a valid `account_id` and matching company fields.
- `contact_id` is the stable contact identifier.
- `persona_fit` is a workshop classification field defined by the targeting and outbound rules.
- Sequence status, lead status, lifecycle stage, engagement, email validity, next step, and CRM notes belong in `02_SOURCE_SYSTEMS/crm_state.csv` or `activities.csv`.

## Relationships

- `contacts.account_id` joins to `accounts.account_id`.
- `02_SOURCE_SYSTEMS/crm_state.csv` contains operational account and contact state.
- `02_SOURCE_SYSTEMS/signals.csv` contains research signals linked to the original scenario accounts.

All records are fictional and safe for workshop use.
